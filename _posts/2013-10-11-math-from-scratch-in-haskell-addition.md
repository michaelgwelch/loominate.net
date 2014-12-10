---
title: 'Math from scratch in Haskell: addition'
author: Michael
layout: post
permalink: /2013/10/11/math-from-scratch-in-haskell-addition/
categories:
  - Uncategorized
---
[Last time][1] I developed a representation of Natural numbers following Eric Lippert's lead. In this post I'll continue to [follow him][2] and implement addition.

<!--more-->

Eric did all the hard work of explaining the recursive algorithm for addition. 

First thing I'll need is an equality operator for Bit. I'll just derive that by adding &#8220;deriving Eq&#8221; to the Bit definition.


{% highlight haskell %}
data Bit = ZeroBit | OneBit deriving Eq
{% endhighlight %}


Here is the implementation of natural addition:

{% highlight haskell linenos %}
natAdd :: Natural -> Natural -> Natural
natAdd n Zero = n
natAdd Zero n = n
natAdd n1@(Nat h1 t1) n2@(Nat h2 t2)
  | h1 == ZeroBit = createNatural (natAdd t1 t2) h2
  | h2 == ZeroBit = createNatural (natAdd t1 t2) h1
  | otherwise = createNatural (natAdd one (natAdd t1 t2)) ZeroBit
{% endhighlight %}


Then like Eric I added the ability to convert a Natural to a string. In Haskell this is done by implementing instance of Show. I do this for both Bit and Natural:

{% highlight haskell linenos %}
instance Show Bit where
  show ZeroBit = "0"
  show OneBit = "1"

instance Show Natural where
  show Zero = "0"
  show (Nat h t) = show t ++ show h
{% endhighlight %}

Finally I added an increment method and exposed the new functions. The whole implementation now looks like this:

{% highlight haskell linenos %}
module Natural (Natural, zero, one, natAdd, natInc) where

data Bit = ZeroBit | OneBit deriving Eq

data Natural = Zero | Nat Bit Natural

zero = Zero
one = Nat OneBit Zero

-- Creates a natural number. Keeps the invariant that
-- Naturals always end with Zero and never end with ZeroBit followed by Zero
createNatural :: Natural -> Bit -> Natural
createNatural Zero ZeroBit = Zero
createNatural Zero OneBit = one
createNatural n b = Nat b n


-- part three: addition and toString (show in Haskell)
-- Also added derived implementation of Eq to Bit above
natAdd :: Natural -> Natural -> Natural
natAdd n Zero = n
natAdd Zero n = n
natAdd n1@(Nat h1 t1) n2@(Nat h2 t2)
  | h1 == ZeroBit = createNatural (natAdd t1 t2) h2
  | h2 == ZeroBit = createNatural (natAdd t1 t2) h1
  | otherwise = createNatural (natAdd one (natAdd t1 t2)) ZeroBit

instance Show Bit where
  show ZeroBit = "0"
  show OneBit = "1"

instance Show Natural where
  show Zero = "0"
  show (Nat h t) = show t ++ show h

-- bug guy meets math from scratch: added increment
natInc :: Natural -> Natural
natInc = natAdd one
{% endhighlight %}

 [1]: http://loominate.net/2013/10/09/math-from-scratch-in-haskell-zero-and-one/ "Math from scratch in Haskell: zero and one"
 [2]: http://ericlippert.com/2013/09/23/math-from-scratch-part-three/
