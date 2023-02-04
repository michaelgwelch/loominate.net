---
title: 'Brainmess: Extract Jump Methods'

layout: post
permalink: /2012/03/19/brainmess-extract-jump-methods/
description:
  - The first steps in refactoring my Brainmess implementation.
keywords:
  - brainmess, brainfuck, clean code, refactoring
categories:
  - software development
tags:
  - brainmess
  - clean code
  - refactoring
---
Today, I'll start to refactor the Brainmess program. In the first [post][1] I gave an "all-in-one" solution. [Next][2] I added some automated tests to give me some confidence that I don't break anything during the process. The last [time][3] that I spoke about Brainmess, I just explained my implementation.

 [1]: {{site.baseurl}}{% post_url 2012-03-13-brainmess %} "Brainmess"
 [2]: {{site.baseurl}}{% post_url 2012-03-14-brainmess-part-2 %} "Brainmess Add Tests"
 [3]: {{site.baseurl}}{% post_url 2012-03-15-brainmess-commentary %} "Brainmess: Commentary"

<!--more-->

Notice that in the switch statement every case, except the last two cases, is one line of code (followed by a `break`). The last two cases are several lines long. These two are prime candidates for <emph>Extract Method</emph>. Why? The first reason is to reduce the length and nesting level of the program. The second is that I suspect these methods which are concerned with finding matching brackets are good candidates for unit testing. So I extract out the `JumpForward` and `JumpBackward` methods. The main method now looks like this:

<pre class="brush: csharp; title: ; notranslate" title="">// skip lines
case '[':
    if (tape[tc] == 0)
    {
        pc = JumpForward(program, pc);
    }
        break;
case ']':
    if (tape[tc] != 0)
    {
        pc = JumpBackward(program, pc);
    }
    break;
// skip lines
</pre>

I think this is already cleaner. The methods make it clear that in one case we are jumping forward, and in the next we are jumping backward. In both cases, the methods scan the program starting from the current position and return the location of the matching bracket.

The methods look like this:

<pre class="brush: csharp; title: ; notranslate" title="">private static int JumpForward(string program, int pc)
{
   int nestLevel = 1;
   while (nestLevel > 0)
   {
       char instruction = program[pc];
       if (instruction == '[')
       {
           nestLevel++;
       }
       else if (instruction == ']')
       {
           nestLevel--;
       }
       pc++;
   }
   return pc;
}

private static int JumpBackward(string program, int pc)
{
   pc -= 2;
   int nestLevel = 1;
   while (nestLevel > 0)
   {
       char instruction = program[pc];
       if (instruction == '[')
       {
           nestLevel--;
       }
       else if (instruction == ']')
       {
           nestLevel++;
       }
       pc--;
   }
   pc++;
   return pc;
}
</pre>

You can see the change I made (and the full files) by visiting my GitHub repository and viewing commit [4b15b4ca][4]. After I made these changes, I ran my tests and found that they still passed.

Now, I'm not totally satisfied with the new methods. The first problem I want to address is that the two methods look almost identical. The main differences between the two is whether we increment or decrement a variable.

The second problem has to do with the `pc -= 2` in the `JumpBackward` method. What is going on there? And why is the `nestLevel` initialized to 1 in both cases?

In the `Run` method we always increment the `pc` variable before executing the instruction. Therefore, when we go to execute the jump instructions the `pc` variable is pointing to the instruction immediately after the jump instruction. In the case of `JumpForward` this means that the `nestLevel` is indeed 1. We are nested one level deep relative to the current jump instruction. In the case of the `JumpBackward` we are in the same position, but only if we back up two instructions.

The third problem with this code, is that the jump instructions have this strange pre-condition that the `pc` variable needs to be positioned 1 after the bracket that caused the jump. That seems odd.

I'm going to create one new method named `FindMatch` that fixes all of these problems.

<pre class="brush: csharp; title: ; notranslate" title="">private static int JumpForward(string program, int pc)
{
   const int increment = 1;
   return FindMatch(program, pc - 1, increment) + 1;
}

private static int JumpBackward(string program, int pc)
{
   const int increment = -1;
   return FindMatch(program, pc - 1, increment);
}

/// <summary>
/// Finds the match for the bracket pointed to by
/// pc in the program. Increment tells the algorithm
/// which way to search.
/// </summary>
private static int FindMatch(string program, int pc, int increment)
{
   int nestLevel = 1;
   pc += increment;
   while (nestLevel > 0)
   {
       char instruction = program[pc];
       if (instruction == '[') nestLevel += increment;
       else if (instruction == ']') nestLevel -= increment;
       pc += increment;
   }
   return pc - increment;
}
</pre>

It solves the first problem because it takes a `increment` variable that indicates which way to search thru the program string. This allows us to have just one method that knows how to find matching strings. (I still don't like this exactly, but I'll talk more about this later.)

It solves the second and third problems by stating the fact that it expects `pc` to point to an actual bracket. This method then finds the matching bracket. Like before we know that the `nestLevel` is 1. This is only true however, because on the next line we either move forward (or backward) to get "inside" of the loop.

I then updated the jump methods to delegate to `FindMatch`. They pass in 1 or -1 as appropriate for the `increment` parameter. In addition they don't pass in the current `pc` value. They pass in `pc - 1` which makes sure we are telling FindMatch to start with a bracket.

This change can be found at commit [abe37577][5]. Again, I ran my tests and they passed.

Now the last refactoring for this post.

I'm going to convert FindMatch into an extension method that can be used on any string, and I'm going to remove the `increment` parameter. This is what the `Run` method looks after the change.

<pre class="brush: csharp; title: ; notranslate" title="">// skip lines
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
// skip lines
</pre>

Why did I remove the `increment` parameter? It didn't make sense to me. The method `FindMatch` should find the the match and determine which way to search. So here is the implementation.

<pre class="brush: csharp; title: ; notranslate" title="">using System;

namespace BrainmessShort
{
    public static class StringExtensions
    {
        /// <summary>
        /// Finds the match for the bracket pointed to by
        /// pc in the program. Increment tells the algorithm
        /// which way to search.
        /// </summary>
        /// <param name="program"></param>
        /// <param name="pc"></param>
        /// <param name="increment"></param>
        /// <returns></returns>
        private static int FindMatch(this string program, int pc, int increment)
        {
            int nestLevel = 1;
            pc += increment;
            while (nestLevel > 0)
            {
                char instruction = program[pc];
                if (instruction == '[') nestLevel += increment;
                else if (instruction == ']') nestLevel -= increment;
                pc += increment;
            }
            return pc - increment;
        }

        public static int FindMatch(this string program, int pc)
        {
            if (program[pc] == '[') return program.FindMatch(pc, 1);
            if (program[pc] == ']') return program.FindMatch(pc, -1);
            throw new ArgumentException("The character at specified location is not a square bracket");

        }
    }
}
</pre>

You can see that there is one public version of `FindMatch` that determines the value
of increment and then delegates to the private one. All the code for this change can be found at commit [abe37577][5].

Finally, I reran all my tests and they passed.


 [4]: https://github.com/michaelgwelch/brainmess/commit/4b15b4caf3eeeaef4da1eba9fcfd78071a8ed51a#diff-1 "Commit"
 [5]: https://github.com/michaelgwelch/brainmess/commit/abe37577309037c5d7c41a7e097293437b56486b "abe37577"
