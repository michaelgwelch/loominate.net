---
title: Brainmess Description

layout: post
permalink: /2012/03/31/brainmess-description/
keywords:
  - brainmess, brainfuck
description:
  - A description of Brainmess
categories:
  - software development
tags:
  - brainmess
math: true
---

It occurred to me that since the [links][1] I've provided so far include a NSFW word, I should take the time to describe the Brainmess language on my blog so readers need not click on those links. This description is obviously not going to be completely original as I've stolen most of it from Mark Chu-Carroll's blog and Wikipedia.

 [1]: {{site.baseurl}}{% post_url 2012-03-13-brainmess %} "Brainmess"
<!--more-->

The Brainmess language has 8 instructions each represented by a single character. Every other character in a Brainmess program is ignored and can be used for comments.

Unlike most high-level languages, there are no named variables in the language. Instead, memory is modeled as a tape with an infinite amount of cells on it. Each cell can hold one number. (In my implementations I default each cell to have an initial value of 0.)

At any given time one and only one of the cells of the tape is the *current* cell. One can think of there being a sensor or tape head that can be positioned over one and only one cell. Six of the 8 instructions make reference to the current cell. It is not possible to read or write any cell but the current cell.

The following table presents the eight instructions.

| Instruction | Name                     | Description                            |
| ----------- | ------------------------ | -------------------------------------- |
| `>`         | *Move Forward*           | Moves the tape head forward one cell.  |
| `<`         | *Move Backward*          | Moves the tape head backward one cell. |
| `+`         | *Increment*              | Increment the current cell by 1.       |
| `-`         | *Decrement*              | Decrement the current cell by 1.       |
| `.`         | *Write*                  | Write the current cell as a character  |
| `,`         | *Read*                   | Read one character into current cell   |
| `[`         | *Test and Jump Forward*  |                                        |
| `]`         | *Test and Jump Backward* |                                        |

I want to provide a little more detail about some of the intructions (espeically the "Test and Jump" instructions).

- *Move Forward* (`>`) Moves the tape head forward one cell. So if the tape head prior to execution is at position $$n$$ of the tape then the tape head after execution will be at position $$n+1$$.
- *Move Backward* (`<`) Moves the tape head backward one cell. So if the tape head prior to execution is at position $$n$$ of the tape then the tape head after execution will be at position $$n-1$$.
- *Write* (`.`)  Treat the value of the current cell as a character and print it to the console.
- *Read* (`,`) Read one character from the console and record its numeric value on the current cell.
- *Test and Jump Forward* (`[`) Test the value of the current cell. If it **is** $$0$$ then move the program ahead to the instruction immediately after the *matching* `]`. (Note the emphasis on the word 'matching'. Nested brackets are supported and a Brainmess program should have balanced brackets.)
- *Test and Jump Backward* (`]`) Test the value of the current cell. If it **is not** $$0$$ then move the program back to the matching `[` instruction.

And that's it! Here is a 'Hello, World' program written in Brainmess. (Remember any character besides the 8 instructions can be used as a comment.)

```text
++++++++        Initialize cell 0 to the number 8
[>+++++++++<-]  While cell 0 is not 0 add 9 to cell 1 and decrement cell 0
>               Move to cell 1 which now has the value of 72
.               Output 'H'
<+++++          Go back to cell 0 and set its value to 5
[>++++++<-]     While cell 0 is not 0 add 6 to cell 1 and decrement cell 0 so cell 1 has value 102
>
>- Move to cell 1 and decrement it so it now has value 101
>
.               Output 'e'
+++++++         Add 7 to cell 1
..              Output 'll' (Adding 7 to 101 makes 108)
+++             Add 3 to cell 1
.               Output 'o' (Adding 3 to 108 makes 111)
<++++++++       Set cell 0 to 8
[>>++++<<-]     While cell 0 is not 0 add 4 to cell 2 which results in 32 in cell 2
>>              Move to cell 2
.               Output ' '
<<              Move to cell 0
++++            Set cell 0 to 4
[>------<-]     Subtracts 24 from cell 1 leaving a value of 87
>               Move to cell 1
.               Output 'W'
<++++           Move to cell 0 and set its value to 4
[>++++++<-]     Add 24 to cell 1 resulting in value 111
>               Move to cell 1
.               Output 'o'
+++             Add 3 to cell 1 resulting in value 114
.               Output 'r'
------          Subtract 6 from cell 1 resulting in value of 108
.               Output 'l'
--------        Subtract 8 from cell 1 resulting in value of 100
.               Output 'd'
>
>- Move to cell 2 and add 1 resulting in value of 33
>
.               Output '!'
```
{: file="hello_world.bm"}

Mark's [post][bf] on this also links to another <a href="http://www.muppetlabs.com/~breadbox/bf/" title="Brainmess">site</a> that gives a very concise description of Brainmess in terms of the C programming language.

[bf]: http://www.goodmath.org/blog/2009/09/06/the-one-the-only-brainfck/ "The One the Only"

Assuming that $$p$$ has been defined to be a `char*`, then the Brainmess instructions can be thought of as the following C expressions:

| Instruction | Implementation  |
| ----------- | --------------- |
| <           | `++p;`          |
| >           | `--p;`          |
| +           | `++(*p);`       |
| -           | `--(*p);`       |
| .           | `putchar(*p);`  |
| ,           | `*p=getchar();` |
| [           | `while(*p) {`   |
| ]           | `}`             |

While this appears to be correct to me, it is actually a little different than the Brainmess specification. Brainmess specifies that the `]` instruction test the current cell. This C implementation doesn't do that. It just jumps back to the beginning of the loop and relies on the fact that the current cell will be tested as part of the `while` condition.
