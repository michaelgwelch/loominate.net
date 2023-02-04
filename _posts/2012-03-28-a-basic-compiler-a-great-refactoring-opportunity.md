---
title: 'A Basic Compiler &#8211; A Great Refactoring Opportunity'

layout: post
permalink: /2012/03/28/a-basic-compiler-a-great-refactoring-opportunity/
categories:
  - software development
tags:
  - basic
  - refactoring
  - structure programming
  - ti-99/4a
---
Years ago (2006) I wrote a [compiler][1] for TI BASIC, the dialect of BASIC that I learned on my TI 99/4A (many many years earlier). This is an &#8220;ancient&#8221; computer language and one of the first that I learned. (I had a few years of experience with Apple Basic on Apple IIe before getting a TI.)

 [1]: https://github.com/michaelgwelch/mbasic99 "TI BASIC Compiler"
<!--more-->

I was reminded of this project tonight when I added my github repositories to my LinkedIn profile. There is now a slightly larger chance that someone might actually see this code. Which leads me to my point: there is a lot of bad code in that repository. I hadn't really seen the &#8220;refactoring light&#8221; at that point in my career and there are large chunks of that code base that are badly in need of it.

The one saving grace is that there are actual tests included with the project. I took code examples from the reference manual and typed them in as separate programs.  I also coded up the expected output from each program in python scripts. I then have a Makefile that compiles each sample program, runs it, and pipes the output to the python script that checks that each line of output matches what is expected.

So this means that I can be fairly confident in refactoring some of this code 6 years later. I hardly know where to start however. I think the best thing to do is to just dive into the `Main` method and starting performing lots of `ExtractMethod`.

## Switching Topics &#8211; About BASIC Badness

Grade school and high school gave me about 9 years of programming in BASIC. I arrived at college with years of programming experience and yet had never heard of a compiler. I recall snickering to myself about how archaic any language must be that requires a compiler (a whole separate tool) before it could be run. I couldn't fathom the purpose of such a tool. (At this time we didn't have the plethora of scripting languages that also have this great feature of my original BASIC.)

I started learning about &#8220;structured&#8221; programming as it was called at the time. I didn't learn any object oriented languages (they were fairly new and not taught). I learned Pascal and the C. We learned not OO but how to use structured constructs. Again, I snickered to myself. What could this terminology mean.

Well, any way, I learned how to program in C and Pascal and Fortran, and then later in life on jobs I learned C++, Visual Basic, and eventually C#, Java and others. I became so comfortable with my new languages and I don't recall there every being a moment when I realized the code I was writing was completely different from BASIC.

Later in life I heard many of the famous quotes by [Edsger W. Dijkstra][2] like

> It is practically impossible to teach good programming to students that have had a prior exposure to BASIC: as potential programmers they are mentally mutilated beyond hope of regeneration.

I didn't really understand what was meant. It had been years since I had studied/used BASIC but couldn't recall it being bad. Well, when I started the compiler project that is the topic of this post I got to become reacquainted with BASIC and see it in a fresh light. It is truly frightening and I wonder how I ever got any program to work. Here is just one small example program (no indentation, no &#8220;structure&#8221;, no white space, no symbolic names for objects or methods to help explain what is going on):

[VB]
100 REM
101 REM
108 DIM STACK$(100)
109 REM
110 GOSUB 1000
130 INPUT &#8220;String: &#8220;:STRING$
140 FOR I = 1 TO LEN(STRING$)
150 CHAR$ = SEG$(STRING$, I, 1)
151 STACKVAL$ = CHAR$
155 LEFTBRACKET = (CHAR$=&#8221;(&#8220;) + (CHAR$=&#8221;[&#8220;) + (CHAR$=&#8221;{&#8220;)
156 RIGHTBRACKET = (CHAR$=&#8221;)&#8221;) + (CHAR$=&#8221;]&#8221;) + (CHAR$=&#8221;}&#8221;)
160 IF LEFTBRACKET THEN 180
170 IF RIGHTBRACKET THEN 190
175 GOTO 250
180 GOSUB 2000
185 GOTO 250
190 GOSUB 3000
195 MATCH = ((STACKVAL$=&#8221;(&#8220;)\*(CHAR$=&#8221;)&#8221;) + (STACKVAL$=&#8221;[&#8220;)\*(CHAR$=&#8221;]&#8221;) + (STACKVAL$=&#8221;{&#8220;)*(CHAR$=&#8221;}&#8221;))
200 IF MATCH=0 THEN 300
250 NEXT I
260 GOSUB 4000
270 IF STACKCOUNT<>0 THEN 300
280 PRINT &#8220;Match&#8221;
290 GOTO 301
300 PRINT &#8220;No match detected at pos &#8220;&STR$(I)
301 INPUT &#8220;Another String (Y or N): &#8220;:AGAIN$
302 IF (AGAIN$=&#8221;Y&#8221;) THEN 110
310 END
999 REM
1000 STACKIDX = -1
1010 STACKVAL$ = &#8220;&#8221;
1020 RETURN
1999 REM
2000 STACKIDX = STACKIDX + 1
2010 STACK$(STACKIDX) = STACKVAL$
2020 RETURN
3000 REM
3010 REM
3020 REM
3030 IF (STACKIDX > -1) THEN 3060
3040 STACKVAL$ = &#8220;&#8221;
3050 GOTO 3080
3060 STACKVAL$ = STACK$(STACKIDX)
3070 STACKIDX = STACKIDX &#8211; 1
3080 RETURN
4000 REM
4010 REM
4020 REM
4030 STACKCOUNT = STACKIDX + 1
4040 RETURN
[/VB]

Actually, this isn't nearly as bad as it was &#8220;back in the day&#8221;. I turned on a little syntax highlighting. Also, before I copied this code I removed the REMARKS and just left placeholders for where they should go. Also, I actually did try to structure this program. Try to see if you can figure out what it is doing and then look at this version with remarks and a little white space.

[VB]

100 REM Checks a string for matching brackets
101 REM &#8216;(&#8216;, &#8216;)', &#8216;[&#8216;, &#8216;]', &#8216;{&#8216;, &#8216;}'

108 DIM STACK$(100)

109 REM Initialize Stack
110 GOSUB 1000
130 INPUT &#8220;String: &#8220;:STRING$

140 FOR I = 1 TO LEN(STRING$)
150 CHAR$ = SEG$(STRING$, I, 1)
151 STACKVAL$ = CHAR$
155 LEFTBRACKET = (CHAR$=&#8221;(&#8220;) + (CHAR$=&#8221;[&#8220;) + (CHAR$=&#8221;{&#8220;)
156 RIGHTBRACKET = (CHAR$=&#8221;)&#8221;) + (CHAR$=&#8221;]&#8221;) + (CHAR$=&#8221;}&#8221;)
160 IF LEFTBRACKET THEN 180
170 IF RIGHTBRACKET THEN 190
175 GOTO 250
180 GOSUB 2000
185 GOTO 250
190 GOSUB 3000
195 MATCH = ((STACKVAL$=&#8221;(&#8220;)\*(CHAR$=&#8221;)&#8221;) + (STACKVAL$=&#8221;[&#8220;)\*(CHAR$=&#8221;]&#8221;) + (STACKVAL$=&#8221;{&#8220;)*(CHAR$=&#8221;}&#8221;))
200 IF MATCH=0 THEN 300
250 NEXT I
260 GOSUB 4000
270 IF STACKCOUNT<>0 THEN 300
280 PRINT &#8220;Match&#8221;
290 GOTO 301
300 PRINT &#8220;No match detected at pos &#8220;&STR$(I)
301 INPUT &#8220;Another String (Y or N): &#8220;:AGAIN$
302 IF (AGAIN$=&#8221;Y&#8221;) THEN 110
310 END

999 REM SUBROUTINE 1000 Initializes the stack to be empty
1000 STACKIDX = -1
1010 STACKVAL$ = &#8220;&#8221;
1020 RETURN

1999 REM This subroutine pushes STACKVAL$ onto stack
2000 STACKIDX = STACKIDX + 1
2010 STACK$(STACKIDX) = STACKVAL$
2020 RETURN

3000 REM This subroutine pops a value off of the stack
3010 REM and puts it into variable STACKVAL$. If the
3020 REM stack is empty then STACKVAL$ will get the empty string
3030 IF (STACKIDX > -1) THEN 3060
3040 STACKVAL$ = &#8220;&#8221;
3050 GOTO 3080
3060 STACKVAL$ = STACK$(STACKIDX)
3070 STACKIDX = STACKIDX &#8211; 1
3080 RETURN

4000 REM This routine updates STACKCOUNT with the number
4010 REM of items in the STACK. STACKCOUNT is only reliable
4020 REM if this subroutine is called before inspecting it.
4030 STACKCOUNT = STACKIDX + 1
4040 RETURN

[/VB]

It was a lot of fun learning to program in TI BASIC, but boy I sure don't miss it.


 [2]: http://en.wikiquote.org/wiki/Programming_languages#BASIC "Wikiquote"
