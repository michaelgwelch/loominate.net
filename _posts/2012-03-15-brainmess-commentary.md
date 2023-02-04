---
title: 'Brainmess: Commentary'

layout: post
permalink: /2012/03/15/brainmess-commentary/
keywords:
  - brainmess, brainfuck
description:
  - This provide commentary on my original implementation of Brainmess. The original name of the language is not safe for work so I use meta tags to include the actual name.
categories:
  - software development
tags:
  - brainmess
---
Today I wanted to step back and explain the implementation of Brainmess that I presented in my first [post][1] on this subject. I recommend you click on the link and have the implementation open in another window as you read this post as I'll be referring to line numbers.

 [1]: {{site.baseurl}}{% post_url 2012-03-13-brainmess %} "Brainmess"

<!--more-->

## Main Method

The `Main` method expects that the path to a script is passed in as the first argument to the program. So the expected usage of this program is to invoke it from the command line with one parameter that is the script to execute:

<pre>> brainmess script.bm</pre>

So in the preceding case, brainmess would attempt to open up a file named `script.bm` and interpret it as a Brainmess program.

The `Main` method simply opens up the file, reads all of its contents, and passes them to the `Run` method.

## Run Method

There are 2 variables used for the program. The first is named `program` and it is a string that contains the entire Brainmess program. As you can see on line 15, this is passed in as a parameter. The second variable used in conjunction with the program is `pc` which is short for program counter. It is the zero-based index that tells us what is the next instruction in the program to "fetch". It is initially set to 0 so that I start fetching the first instruction.

There are also 2 variables (lines 18 and 19) that are used for the tape (the tape serves as our memory). The first is named `tape` and it is an array of integers. The variable `tc` is our "tape counter". It is the zero-based index into the tape. Six of the Brainmess instructions make reference to the current cell on the tape, and `tc` tells us what the current cell is. I initialize it to the middle of the array so that it is legal for a program to move in either direction. (This is not a requirement for Brainmess interpreters.)

The `Run` method is a big while loop. It runs a "fetch/execute"cycle. That is, on each iteration of the loop it fetches the next instruction from the program and then executes it. On lines 23 and 24 is where the fetch happens. During a fetch it reads the next instruction and then increment the program counter so that it is ready for the next iteration. This loop continues until it reaches the end of the program.

On lines 25-73 I have a large switch statement that switches on the current instruction and that has a case for every valid instruction character.

Lines 27 and 30 are the cases for the "Move Forward" and "Move Backward" instructions. The Move Forward and Move Backward instructions tell us to move the tape forward or backward. These are implemented by simply incrementing the tape counter (for Move Forward) or decrementing the tape counter (for Move Backward).

Lines 33 and 36 are the cases for the "Increment" and "Decrement" instructions. These instructions mean that the value on the current cell of the tape should be incremented or decremented. So the implementation is again very simple. This is implemented by incrementing or decrementing the cell at the index pointed to be the value of `tc`.

Line 39 is the "Output" instruction case. For this instruction the interpreter must get the value from the current cell, treat it as the ascii value of a character, and output that character. I chose to do this by writing it to the console.

Line 42 is the "Input" instruction case and it is the "opposite" of the "Output" instruction. For this the interpreter must read in one character and write the ascii value of that character (as an integer) to the current cell of the tape. I chose to read the input from the console.

Line 45 is the "Test and Jump Forward" instruction. This instruction must look at the value of the current cell of the tape and if it is 0 jump to the instruction immediately following the matching &#8216;]' character, otherwise it does nothing. To implement the jump I use the `nestLevel` variable defined on line 20. This variable keeps track of how deeply nested the program counter is inside of loops relative to the current instruction. I initially set this to 1 since it is inside of one loop. I then start moving the program counter forward. Every time I encounter a &#8216;[&#8216; character I must increase the nest level by 1. Every time I encounter a &#8216;]' character I can decrease the nest level by 1. When the nest level reaches 0, I know that I've exited the loop and that the program counter is now immediately following the matching &#8216;]' character.

Line 58 is the "Test and Jump Backward" instruction. This instruction must look at the value of the current cell of the tape and if it is NOT 0 jump backward to the matching &#8216;[&#8216; character, otherwise it does nothing. The algorithm is the same except we are searching in the reverse order and that as we move to the left every &#8216;[&#8216; character decreases the nest level and every &#8216;]' increases the nest level. Also, to initialize the algorithm correctly we must on line 61 decrease the program counter by 2. This moves the pc to one character before the &#8216;]' instruction and guarantees that the nest level starts at 1.

Every other character is treated as a "No Operation" instruction.

That is the complete program. I hope that it is clear what is going on. Please leave questions below in the comments.


