---
title: 'Brainmess: Extract Tape Class (Cleanup)'
author: Michael
layout: post
permalink: /2012/03/31/brainmess-extract-tape-class-cleanup/
categories:
  - Uncategorized
---
This is just a quick update. My last [refactoring][1] on Brainmess was to extract out the tape class. While reviewing that code today, I noticed some artifacts left over from the refactoring process. These were cleaned up in commit [836a96][2]

<!--more-->

Four of the methods: `MoveForward`, `MoveBackward`, `Increment`, `Decrement` still had return values that were not needed. For example before we extracted out the tape class, this is what the `MoveForward` method looked like:

<pre class="brush: csharp; title: ; notranslate" title="">int MoveForward()
{
    return tc++;
}
</pre>

and this is how it was called from the `Run` method:

<pre class="brush: csharp; title: ; notranslate" title="">switch(instruction)
// ... snip
{
case '&gt;': 
    MoveForward();
    break;

// snip...
</pre>

The `MoveForward` method and the <var>tc</var> variable were then moved into the `Tape` class. You can see however that there is no need for the `MoveForward` method to return anything. It can be simplified to the simple expression `tc++`.

Similar changes can be made for the other three methods so that we get this for the Tape class.

<pre class="brush: csharp; title: ; notranslate" title="">public class Tape
{
    private readonly int[] tape = new int[5000];
    private int tc = 2500;
    
    public void MoveForward()
    {
        tc++;
    }
    
    public void MoveBackward()
    {
        tc--;
    }
    
    public void Increment()
    {
        tape[tc]++;
    }
    
    public void Decrement()
    {
        tape[tc]--;
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

 [1]: http://www.loominate.net/2012/03/23/brainmess-extract-tape-class/ "Brainmess: Extract Tape Class"
 [2]: http://github.com/michaelgwelch/brainmess/commit/836a96877b7ffa3b04a75d73f66063dfef9dcbc2 "Cleanup Tape Class"