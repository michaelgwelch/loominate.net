---
title: 'Brainmess: Extract Tape Class'

layout: post
permalink: /2012/03/23/brainmess-extract-tape-class/
description:
  - Continue refactoring my Brainmess implementation.
keywords:
  - brainmess, brainfuck, refactoring
categories:
  - software development
tags:
  - brainmess
  - refactoring
---
Last time I worked on extracting out the methods related to fetching instructions and jump instructions into a `Program` class. I'm going to use a similar pattern today to extract out methods related to the tape. Currently the tape is implemented as an array of integers and a tape counter. I want to replace all of that with a `Tape` class.

<!--more-->

## Extract out tape methods

First I will extract out all the tape related methods and a property. (See commit [746f4e3][1] for extract method details.)

<pre class="brush: csharp; title: ; notranslate" title="">int MoveForward()
{
    return tc++;
}

int MoveBackward()
{
    return tc--;
}

int Increment()
{
return tape[tc]++;
}

int Decrement()
{
    return tape[tc]--;
}

int Current
{
    get
    {
        return tape[tc];
    }
    set
    {
        tape[tc] = value;
    }
}
</pre>

This leaves our `Run` method looking like this:

<pre class="brush: csharp; title: ; notranslate" title="">public void Run()
{
    while(!_program.EndOfProgram)
    {
        char instruction = _program.Fetch();
        switch(instruction)
        {
            case '>':
                MoveForward();
                break;
            case '<':
                MoveBackward();
                break;
            case '+':
                Increment();
                break;
            case '-':
                Decrement();
                break;
            case '.':
                Console.Write((char)Current);
                break;
            case ',':
                Current = Console.Read();
                break;
            case '[':
                if (Current == 0)
                {
                    _program.JumpForward();
                }
                break;
            case ']':
                if (Current != 0)
                {
                    _program.JumpBackward();
                }
                break;
        }
    }
}
</pre>

I rerun all my tests and see that they still all pass.

## Extract Tape Class

These methods along with the `tape` and `tc` variables can be moved into a new `Tape` class. (See commit [c75cc6][2] for the details of this refactoring.)

Here is the `Tape` class:

<pre class="brush: csharp; title: ; notranslate" title="">public class Tape
{
    private readonly int[] tape = new int[5000];
    private int tc = 2500;

    public int MoveForward()
    {
        return tc++;
    }

    public int MoveBackward()
    {
        return tc--;
    }

    public int Increment()
    {
        return tape[tc]++;
    }

    public int Decrement()
    {
        return tape[tc]--;
    }

    public int Current
    {
        get
        {
            return tape[tc];
        }
        set
        {
            tape[tc] = value;
        }
    }
}
</pre>

Leaving the `Run` method looking like this.

<pre class="brush: csharp; title: ; notranslate" title="">public void Run()
{
    while(!_program.EndOfProgram)
    {
        char instruction = _program.Fetch();
        switch(instruction)
        {
            case '>':
                _tape.MoveForward();
                break;
            case '<':
                _tape.MoveBackward();
                break;
            case '+':
                _tape.Increment();
                break;
            case '-':
                _tape.Decrement();
                break;
            case '.':
                Console.Write((char)_tape.Current);
                break;
            case ',':
                _tape.Current = Console.Read();
                break;
            case '[':
                if (_tape.Current == 0)
                {
                    _program.JumpForward();
                }
                break;
            case ']':
                if (_tape.Current != 0)
                {
                    _program.JumpBackward();
                }
                break;
        }
    }
}
</pre>

I think this change makes `Run` read a little better than the version at the top of this post. This is because now each access of the tape is explicit because of the use of the `_tape` variable.

## Benefits

Like last time we get several benefits out of this refactoring: testability, encapsulation, readability.

All the access to the tape is now in one class and has no dependencies on anything else. We can add some methods (like "LoadState" and "GetState") that allow us to create a tape in any state we want for a test setup. Then we can execute one of its methods and use GetState to check our results.

We may also now choose to change our tape implementation. I actually prefer the use of a linked list for the tape because it makes it easy to continue to add cells on either end of the tape. We can go ahead and make this sort of change and not have to worry about how it affects the rest of the program.

Finally, I like how the new `Run` method reads. Again, the variable and method names make the code "self documenting". We know what each branch of the `switch` statement is doing without having to worry about the details. This makes it easier for us to read and it certainly makes it easier for a code reviewer or a future maintainer of the code.

Notice how easy it is to tell which instructions affect the program and which affect the tape. Also, notice how the different "components" are easier to identify.

 [1]: https://github.com/michaelgwelch/brainmess/commit/746f4e3bbdd869b0863e2b18fe63423d59170152
 [2]: https://github.com/michaelgwelch/brainmess/commit/c75cc6a8cefaca42c66570f7bf49f6e16846dea6
