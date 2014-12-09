---
title: 'Math from scratch in Haskell: zero and one'
author: Michael
layout: post
permalink: /2013/10/09/math-from-scratch-in-haskell-zero-and-one/
categories:
  - Uncategorized
tags:
  - Haskell
  - math
  - software development
---
I&#8217;ve been following a set of posts written by [Eric Lippert][1] where he is implementing arbitrary size Naturals and the corresponding arithmetic in C#. As I follow along I&#8217;m writing an implementation in Haskell (one that I hope is idiomatic to Haskell).  
<!--more-->

  
In [part one][2] and [part two][3] of the series he develops a representation of Naturals and exposes two public members: Zero and One. Read those two posts to understand his implementation.

Here is my implementation

<pre class="brush: plain; title: ; notranslate" title="">module Natural (Natural, zero, one) where

data Bit = ZeroBit | OneBit 

data Natural = Zero | Nat Bit Natural 

zero = Zero
one = Nat OneBit Zero

-- Creates a natural number. Keeps the invariant that 
-- Naturals always end with Zero and never end with 
-- a ZeroBit followed by Zero
createNatural :: Natural -&gt; Bit -&gt; Natural
createNatural Zero ZeroBit = Zero
createNatural Zero OneBit = one
createNatural n b = Nat b n
</pre>

On line 1 we expose the new type and two public values: zero and one.

I define two new types: Bit and Natural. The definition of Bit is straightforward. I simply define two constructors: ZeroBit and OneBit. Likewise the definition of Natural follows directly from the recursive definition given by Eric. I can construct a Natural by using the Zero constructor or by using the Nat constructor (which adds a new least significant bit onto an existing Natural).

Like Eric&#8217;s implementation I use a helper function to maintain the invariant he mentions that a Natural never ends in a ZeroBit followed by a Zero.

The definitions of zero and one are straightforward as well. That&#8217;s it. 

I&#8217;ll continue to add posts that follow Eric&#8217;s posts. The next post will show how to implement addition.

 [1]: http://ericlippert.com
 [2]: http://ericlippert.com/2013/09/16/math-from-scratch-part-one/
 [3]: http://ericlippert.com/2013/09/19/math-from-scratch-part-two/