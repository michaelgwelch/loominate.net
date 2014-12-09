---
title: Brainmess
author: Michael
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
I was introduced to a language with a NSFW name a few years ago. Mark Chu-Carroll [posted][1] about it. It is also documented at [Wikipedia][2] (Note, the name of the article I linked to is NSFW due to language). [Updated 3/31/2012: I&#8217;ve now described [Brainmess][3] on my own blog so one need not click the links above while at work.]

I&#8217;ve sanitized the name and used it as the basis of a simple programming assignment for use in study groups at work.

<!--more-->

I&#8217;ve run study groups on &#8220;clean code&#8221;, refactoring and unit testing. In each of these groups, it&#8217;s useful to have some programming assignment as a basis for learning the topic. I ask developers to write an interpreter for Brainmess programs. It&#8217;s a rather simple program to write. It takes most developers just a few hours (or less) to complete. But I ask them to think about how they would test it. I ask them to think about maintainability/readability.

I&#8217;ve actually implemented the Brainmess interpreter many different ways in 4 different languages so far (Java, C#, Haskell, and C). I try different things so that I can consider the questions of readability and testability. 

In this post I&#8217;ll present the quick and dirty all in one method implementation and in future posts I&#8217;ll discuss refactoring, readability and testability.

This project (along with other implementations) can be downloaded from [Brainmess][4] (github). Look for the BrainmessShort project.

Here it is all in one method (except for the reading from a file):

<pre class="brush: csharp; title: ; notranslate" title="">using System;
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
            while(pc &lt; program.Length)
            {
                char instruction = program[pc];
                pc++;
                switch(instruction)
                {
                case '&gt;': 
                    tc++;
                    break;
                case '&lt;':
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
                        while(nestLevel &gt; 0)
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
                        while(nestLevel &gt; 0)
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
</pre>

So I&#8217;ve run a couple of Brainmess scripts thru this interpreter and it seems to work. So what (if anything) is wrong with this implementation? In some sense there is nothing wrong with it. It works and if you take some time to review it, it&#8217;s not to hard to understand.

In another sense there is a lot wrong with it. 

**It doesn&#8217;t convey any design.** What do I mean by that? It&#8217;s not obvious what the different components of the solution are. There are at least 4 components that I normally call out in a more verbose solution: Tape, Program, Input, Output. They are all in this solution, but they are all thrown together and their relationships (if any) to one another are not clear. 

Related to this we have &#8220;data clumps&#8221;. The `tape` and `tc` variables are always used together. But they are never used with other variables. Same with `program` and `pc`. (Hint: It&#8217;s almost as if they should be their own objects). 

**Not unit testable.** How do you test this? Well, you can do like I did and run some scripts thru it. I actually did pretty good. I wrote this version from scratch with only one defect in it (as far as I know &#8211; since it&#8217;s not unit testable). The problem with running scripts is that they may not exercise every scenario. For example, can I guarantee that after a &#8220;Test and Jump Backward&#8221; instruction that the program counter is currently pointing at the matching &#8216;[&#8216; character? Can I guarantee that nested loops work correctly? 

What we often want to do is test the individual components in a controlled environment with a series of automated unit tests. There are several issues when it comes time to try to write unit tests for this. As stated before the components are not isolated. The are all linked together and there are no &#8220;access points&#8221;. In addition, this program is hard-coded to the console for I/O so we can&#8217;t even control the unit test environment without manually watching the tests run.

How do we test the logic of our jump instructions? That code is really the only code that isn&#8217;t completely trivial. But it&#8217;s embedded within our `Run` method where we can&#8217;t get at it to write unit tests.

**Code duplication.** The &#8216;[&#8216; and &#8216;]&#8217; jump instructions execute code that looks very similar but not quite the same. It&#8217;d be nice if we could find a way to not only test that code, but simplify it to just one method (if possible).

**Readability** This is controversial. I know many developers probably think that in terms of readability this is the absolute best version you could write. All of the code is in one place so you can examine it all at once to figure out what it is doing. I&#8217;m not one of those developers. I don&#8217;t like to have to keep track of 4 or 5 variables within the context of 2 nested loops and a switch statement. I&#8217;m very familiar with this logic after writing it so many times and I&#8217;d still say that I prefer to at least extract a few methods. I prefer the creation of useful abstractions that keep track of separate details for me, so that when I&#8217;m looking at one bit of code and can completely forget about the details of another bit of code.

I do agree that this example is simple enough that you don&#8217;t need to &#8220;overdo&#8221; it with abstractions. However, the point of this assignment is that it is a small enough example that you CAN easily play around with different abstractions and testing strategies.

In future posts I&#8217;ll refactor this code. Perhaps I&#8217;ll eventually take it too far just to see where the limits of are.

 [1]: http://scienceblogs.com/goodmath/2009/09/the_one_the_only_brainfck.php
 [2]: http://en.wikipedia.org/wiki/Brainfuck
 [3]: http://www.loominate.net/2012/03/31/brainmess-description/ "Brainmess Description"
 [4]: https://github.com/michaelgwelch/brainmess "GitHub Brainmess"