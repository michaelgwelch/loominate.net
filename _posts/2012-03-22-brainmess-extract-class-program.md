---
title: 'Brainmess: Extract Class Program'
author: Michael
layout: post
permalink: /2012/03/22/brainmess-extract-class-program/
description:
  - More refactoring of my Brainmess implementation.
keywords:
  - brainmess, brainfuck, refactoring
categories:
  - software development
tags:
  - brainmess
  - refactoring
---
Today, I'd like to address the issue of &#8220;data clumping&#8221; in the [original][1] implementation of Brainmess. (See all previous posts: [testing][2], [explanation][3] and [extract methods][4].)

<!--more-->

As was pointed out in an earlier post, the variables `program` and `pc` are generally used together, but not used with other variables. This suggests a tighter relationship between these two. Because of this, I want to extract them into their own class named `Program`.

We'll be doing this in several steps. The first one is to convert the `Run` method into an instance method. Why? Because, we'll be converting `program` and `pc` into member variables and we don't want them to be static variables. Why? Because static variables don't get &#8220;re-initialized&#8221; in between runs. I was reminded of this the hard way. The first time I started this refactoring I just converted them to static variables and ran my tests. Several of them failed because the `pc` variable was not reset back to 0 after every run. This reminded me that these should be instance variables.

## Convert local variables to member variables

So I created a constructor and converted them to instance variables. I left Main as a static method because it needs to be as its the entry point of our program. (See [commit 51f433][5] for all the details.)

<pre class="brush: csharp; title: ; notranslate" title="">public class Brainmess
{
    private readonly string program;
    private int pc = 0;
    private readonly int[] tape = new int[5000];
    private int tc = 2500;
    public Brainmess(string program) 
    {
        this.program = program;
    }
    
    public static void Main(string[] args)
    {       
        var reader = File.OpenText(args[0]);
        new Brainmess(reader.ReadToEnd()).Run();
        reader.Close();
    }
    
    public void Run() 
    {
        while(pc < program.Length)
        {
            char instruction = program[pc];
            pc++;
            switch(instruction)
            {
            case '>': 
                tc++;
                break;
            case '<':
                tc--;
                break;
            case '+':
                tape[tc]++;
                break;
            case '-':
                tape[tc]--;
                break;
            case '.':
                Console.Write((char)tape[tc]);
                break;
            case ',':
                tape[tc] = Console.Read();
                break;
            case '[':
                if (tape[tc] == 0)
                {
                    pc = program.FindMatch(pc - 1) + 1;
                }
                    break;
            case ']':
                if (tape[tc] != 0)
                {
                    pc = program.FindMatch(pc - 1);
                }
                break;
            }
        }
    }
}
</pre>

## Extract program related methods

Now I'm going to perform a couple of &#8220;Extract Methods&#8221; on every location in which `program` or `pc` are used. In this way I'll be encapsulating all access to these variables.

Here are the four new methods.

<pre class="brush: csharp; title: ; notranslate" title="">char Fetch()
{
    var instruction = program[pc];
    pc++;
    return instruction;
}

void JumpForward()
{
    pc = program.FindMatch(pc - 1) + 1;
}

void JumpBackward()
{
    pc = program.FindMatch(pc - 1);
}

bool EndOfProgram
{
    get
    {
        return (pc >= program.Length);
    }
}
</pre>

And this is the new Run method:

<pre class="brush: csharp; title: ; notranslate" title="">public void Run() 
{
    while(!EndOfProgram)
    {
        char instruction = Fetch();
        switch(instruction)
        {
        case '>': 
            tc++;
            break;
        case '<':
            tc--;
            break;
        case '+':
            tape[tc]++;
            break;
        case '-':
            tape[tc]--;
            break;
        case '.':
            Console.Write((char)tape[tc]);
            break;
        case ',':
            tape[tc] = Console.Read();
            break;
        case '[':
            if (tape[tc] == 0)
            {
                JumpForward();
            }
                break;
        case ']':
            if (tape[tc] != 0)
            {
                JumpBackward();
            }
            break;
        }
    }
}
</pre>

You can see that `program` and `pc` are no longer mentioned in the `Run` method. I goofed up and it took me three commits to get this refactoring complete. If you want to see the details see [dbad7a][6], [8f6ee0][7] and [299007][8]. After extracting out these methods, I checked to make sure all my tests still pass.

## Extract Program Class

Now we can go ahead and extract out the new methods and related variables into their own class: `Program`. See commit [356b6b][9] for details.

<pre class="brush: csharp; title: ; notranslate" title="">using System;

namespace BrainmessShort
{
    public class Program
    {
        private readonly string program;
        private int pc;
        
        public Program(string program)
        {
            this.program = program;
            pc = 0;
        }
        
        public bool EndOfProgram
        {
            get
            {
                return pc >= program.Length;
            }
        }
        
        public char Fetch()
        {
            var instruction = program[pc];
            pc++;
            return instruction;
        }

        public void JumpForward()
        {
            pc = program.FindMatch(pc - 1) + 1;
        }

        public void JumpBackward()
        {
            pc = program.FindMatch(pc - 1);
        }
    }
}
</pre>

This is what our `Brainmess` class now looks like. It delegates all program related activities to the `_program` object which is an instance of `Program`.

<pre class="brush: csharp; title: ; notranslate" title="">using System;
using System.IO;

namespace BrainmessShort
{
   public class Brainmess
   {
        private readonly Program _program;
        
        private readonly int[] tape = new int[5000];
        private int tc = 2500;
        public Brainmess(string programString) 
        {
            _program = new Program(programString);
        }
        
        public static void Main(string[] args)
        {       
            var reader = File.OpenText(args[0]);
            new Brainmess(reader.ReadToEnd()).Run();
            reader.Close();
        }
        
        public void Run() 
        {
            while(!_program.EndOfProgram)
            {
                char instruction = _program.Fetch();
                switch(instruction)
                {
                case '>': 
                    tc++;
                    break;
                case '<':
                    tc--;
                    break;
                case '+':
                    tape[tc]++;
                    break;
                case '-':
                    tape[tc]--;
                    break;
                case '.':
                    Console.Write((char)tape[tc]);
                    break;
                case ',':
                    tape[tc] = Console.Read();
                    break;
                case '[':
                    if (tape[tc] == 0)
                    {
                        _program.JumpForward();
                    }
                        break;
                case ']':
                    if (tape[tc] != 0)
                    {
                        _program.JumpBackward();
                    }
                    break;
                }
            }
        }
   }
}
</pre>

## What was the purpose of all this?

First let me remind you that I realize that this refactoring is not necessarily needed for such a small program as Brainmess. However, I did gain something. I now have a reusable class named `Program` whose functionality I can completely unit test independently of anything else in the program. Or at least I can if I add one small property and a static factory method to the class:

<pre class="brush: csharp; title: ; notranslate" title="">public int ProgramCounter
{
    get
    {
        return pc;
    }
}

public static Program LoadState(string programString, int pc)
{
    Program program = new Program(programString);
    program.pc = pc;
    return program;
}
</pre>

With these changes, I can now test that the `Fetch`, `JumpForward` and `JumpBackward` methods all work as expected. The `LoadState` method allows me to create a `Program` instance in any state. The normal constructor always starts the program counter at the beginning. `LoadState` let's me choose any starting location for it to set up any test environment I want. The `ProgramCounter` property allows me to check the state of the program after a method is executed.

Here are some examples of some tests I wrote. Note, the methods in this class only have one path and therefore really only require one unit test each. The real &#8220;hard&#8221; logic for the Jump instructions is buried in the `FindMatch` method and therefore that method has more tests.

<pre class="brush: csharp; title: ; notranslate" title="">using System;
using NUnit.Framework;
namespace BrainmessShort
{
    [TestFixture]
    public class ProgramStreamTests
    {
        // White box testing. Each method only needs one test because there are no alternative paths.
        // The "hard" testing is done for StringExtensions.FindMatch
        
        
        [Test]
        public void JumpForward()
        {
            // Arrange
            //                                   0123456789
            Program program = Program.LoadState("++[     ]   ", 3);

            // Act
            program.JumpForward();

            // Assert
            Assert.AreEqual(9, program.ProgramCounter);

        }

        [Test]
        public void JumpBackward()
        {
            // Arrange
            //                                   0123456789
            Program program = Program.LoadState("++[     ]   ", 9);

            // Act
            program.JumpBackward();

            // Assert
            Assert.AreEqual(2, program.ProgramCounter);
        }

        [Test]
        public void Fetch__ShouldReturnNextInstruction()
        {
            // Arrange
            //                                   0123456789
            Program program = Program.LoadState("++[>>.>>>abc]   ", 4);

            // Act
            var instruction = program.Fetch();

            // Assert
            Assert.AreEqual('>', instruction);
        }

    }

}
</pre>

Another benefit is that I can now change my implementation of `Program` at any time and not have to worry about breaking something in the rest of the Brainmess program. What is an example of a change? Well consider that we might want to run very large Brainmess programs and we don't want to load the whole program into memory. Then I could modify `Program` to read directly from a `Stream`, like perhaps a `FileStream`.

Finally, the new methods &#8220;document&#8221; the code. In the `Run` method, you don't need any comments to explain one each program related activity is doing. The method name tells you what is going on. I think this greatly increases the readability.

Next time, we'll do a similar thing with the tape related statements.

 [1]: http://www.loominate.net/2012/03/13/brainmess/ "Brainmess"
 [2]: http://www.loominate.net/2012/03/14/brainmess-part-2/ "Brainmess: Part 2"
 [3]: http://www.loominate.net/2012/03/15/brainmess-commentary/ "Brainmess: Commentary"
 [4]: http://www.loominate.net/2012/03/19/brainmess-extract-jump-methods/ "Brainmess: Extract Jump Methods"
 [5]: https://github.com/michaelgwelch/brainmess/commit/51f4330605395edd0d023215f873e66dbe2edb91 "Convert to Instance Methods and Fields"
 [6]: https://github.com/michaelgwelch/brainmess/commit/dbad7a86b5f5e850e11f897364d876c01ec8ae47
 [7]: https://github.com/michaelgwelch/brainmess/commit/8f6ee036182c9584d639f58456476aefb62b8701
 [8]: https://github.com/michaelgwelch/brainmess/commit/29900746cae42b30ed49deff6a0ef846efa9dfb6
 [9]: https://github.com/michaelgwelch/brainmess/commit/356b6b6b7ef46b4b32a20183fdc3c699efa9863b
