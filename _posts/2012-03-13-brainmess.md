---
title: Brainmess

layout: post
permalink: /2012/03/13/brainmess/
keywords:
  - brainmess, brainfuck, refactoring, unit testing, clean code
categories:
  - software development
tags:
  - brainmess
  - clean code
  - readability
  - refactoring
  - unit testing
---
I was introduced to a language with a NSFW name a few years ago. Mark Chu-Carroll [posted][1] about it. It is also documented at [Wikipedia][2] (Note, the name of the article I linked to is NSFW due to language). [Updated 3/31/2012: I've now described [Brainmess][3] on my own blog so one need not click the links above while at work.]

 [1]: http://scienceblogs.com/goodmath/2009/09/the_one_the_only_brainfck.php
 [2]: http://en.wikipedia.org/wiki/Brainfuck
 [3]: {{site.baseurl}}{% post_url 2012-03-31-brainmess-description %} "Brainmess Description"

<!--more-->

I've sanitized the name and used it as the basis of a simple programming assignment for use in study groups at work.


I've run study groups on "clean code", refactoring and unit testing. In each of these groups, it's useful to have some programming assignment as a basis for learning the topic. I ask developers to write an interpreter for Brainmess programs. It's a rather simple program to write. It takes most developers just a few hours (or less) to complete. But I ask them to think about how they would test it. I ask them to think about maintainability/readability.

I've actually implemented the Brainmess interpreter many different ways in 4 different languages so far (Java, C#, Haskell, and C). I try different things so that I can consider the questions of readability and testability.

In this post I'll present the quick and dirty all in one method implementation and in future posts I'll discuss refactoring, readability and testability.

This project (along with other implementations) can be downloaded from [Brainmess][4] (github). Look for the BrainmessShort project.

Here it is all in one method (except for the reading from a file):

{% highlight csharp linenos %}
using System;
using System.IO;

namespace BrainmessShort
{
   public class Brainmess
   {
        public static void Main(string[] args)
        {
            var reader = File.OpenText(args[0]);
            Run(reader.ReadToEnd());
            reader.Close();
        }

        public static void Run(string program)
        {
            int pc = 0;
            int[] tape = new int[5000];
            int tc = 2500;
            int nestLevel;
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
                        nestLevel = 1;
                        while(nestLevel > 0)
                        {
                            instruction = program[pc];
                            if (instruction == '[') nestLevel++;
                            else if (instruction == ']') nestLevel--;
                            pc++;
                        }
                    }
                    break;
                case ']':
                    if (tape[tc] != 0)
                    {
                        pc -= 2;
                        nestLevel = 1;
                        while(nestLevel > 0)
                        {
                            instruction = program[pc];
                            if (instruction == '[') nestLevel--;
                            else if (instruction == ']') nestLevel++;
                            pc--;
                        }
                        pc++;
                    }
                    break;
                }
            }
        }
    }
}
{% endhighlight %}

So I've run a couple of Brainmess scripts thru this interpreter and it seems to work. So what (if anything) is wrong with this implementation? In some sense there is nothing wrong with it. It works and if you take some time to review it, it's not to hard to understand.

In another sense there is a lot wrong with it.

**It doesn't convey any design.** What do I mean by that? It's not obvious what the different components of the solution are. There are at least 4 components that I normally call out in a more verbose solution: Tape, Program, Input, Output. They are all in this solution, but they are all thrown together and their relationships (if any) to one another are not clear.

Related to this we have "data clumps". The `tape` and `tc` variables are always used together. But they are never used with other variables. Same with `program` and `pc`. (Hint: It's almost as if they should be their own objects).

**Not unit testable.** How do you test this? Well, you can do like I did and run some scripts thru it. I actually did pretty good. I wrote this version from scratch with only one defect in it (as far as I know&#8212;since it's not unit testable). The problem with running scripts is that they may not exercise every scenario. For example, can I guarantee that after a "Test and Jump Backward" instruction that the program counter is currently pointing at the matching `[` character? Can I guarantee that nested loops work correctly?

What we often want to do is test the individual components in a controlled environment with a series of automated unit tests. There are several issues when it comes time to try to write unit tests for this. As stated before the components are not isolated. The are all linked together and there are no "access points". In addition, this program is hard-coded to the console for I/O so we can't even control the unit test environment without manually watching the tests run.

How do we test the logic of our jump instructions? That code is really the only code that isn't completely trivial. But it's embedded within our `Run` method where we can't get at it to write unit tests.

**Code duplication.** The `[` and `]` jump instructions execute code that looks very similar but not quite the same. It'd be nice if we could find a way to not only test that code, but simplify it to just one method (if possible).

**Readability** This is controversial. I know many developers probably think that in terms of readability this is the absolute best version you could write. All of the code is in one place so you can examine it all at once to figure out what it is doing. I'm not one of those developers. I don't like to have to keep track of 4 or 5 variables within the context of 2 nested loops and a switch statement. I'm very familiar with this logic after writing it so many times and I'd still say that I prefer to at least extract a few methods. I prefer the creation of useful abstractions that keep track of separate details for me, so that when I'm looking at one bit of code and can completely forget about the details of another bit of code.

I do agree that this example is simple enough that you don't need to "overdo" it with abstractions. However, the point of this assignment is that it is a small enough example that you CAN easily play around with different abstractions and testing strategies.

In future posts I'll refactor this code. Perhaps I'll eventually take it too far just to see where the limits of are.


 [4]: https://github.com/michaelgwelch/brainmess "GitHub Brainmess"
