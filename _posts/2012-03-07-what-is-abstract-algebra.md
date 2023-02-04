---
title: What is Abstract Algebra?

layout: post
permalink: /2012/03/07/what-is-abstract-algebra/
math: true
categories:
  - mathematics
tags:
  - abstract algebra
  - modern algebra
---
I've recently begun a study of abstract algebra (also known as modern algebra) using the text[A Book of Abstract Algebra][1] by Charles C. Pinter.

 [1]: http://www.amazon.com/Book-Abstract-Algebra-Edition-Mathematics/dp/0486474178/ref=sr_1_1?ie=UTF8&qid=1330805123&sr=8-1
<!--more-->

I'll give you a brief introduction to abstract algebra by way of an example. I will write two simple proofs. The first is a proof about addition of real numbers. The second is a proof about the exclusive or operation on Boolean[^f1] values. The proofs will attempt to prove similar looking things, however, the initial proofs will look completely different from one another. Then I'll identify some common traits about the two "[algebraic structures][2]" and show you how the same proof can be applied to both problems. This is one of the things abstract algebra is about: identifying common traits between different algebraic structures.


## Addition of Reals

Let's consider the addition operation on the set of real numbers and prove the if
 $$ A + B = A + C $$  then  $$ B = C $$. This can be accomplished in several simple steps:

$$
\begin{align}
(A + B) &= (A + C) && \text{Given} \tag{1} \\
-A + (A + B) &= -A + (A + C) \tag{2} \\
(-A + A) + B &= (-A + A) + C && \text{Associativity (2)} \tag{3} \\
0 + B &= 0 + C && \tag{4} \\
B &= C \tag{5}
\end{align}
$$

## Exclusive Or

Now let's consider the operation $$\oplus$$ ([exclusive or][3]) on the [Boolean][4] values and prove that $$A \oplus B = A \oplus C$$ implies $$B = C$$. I hope you can see the structure of this proposition is identical to the previous one.

I will prove this statement by [case analysis][5] and the use of the truth table for $$ \oplus$$ (I use the [symbol][6] $$\top$$ for True and the symbol $$\bot$$ for False.):

$$
\begin{array}{cc|c}
  X   &   Y  &  X \oplus Y \\  \hline
 \bot & \bot & \bot \\
 \bot & \top & \top \\
 \top & \bot & \top \\
 \top & \top & \bot \\
\end{array}
$$

To prove this we must consider every case. Let us set $$R\ = A \oplus B = A \oplus C$$. We must consider that $$ A$$ can be $$ \top$$ or $$ \bot$$ and that $$R$$ can be $$\top$$ or $$\bot$$. So in total we have four cases to prove:

  1. First Case: $$ A = \bot$$ and $$R = \bot$$. By substituting the value of $$ A$$ we get $$ \bot \oplus B = \bot \oplus C = \bot$$. By consulting the truth table we can see that this can only be true if $$ B = \bot$$ and $$ C = \bot$$. Therefore $$ B = C$$.
  2. Second Case: $$ A = \bot$$ and $$R = \top$$. By substituting the value of $$ A$$ we get $$ \bot \oplus B = \bot \oplus C = \top$$. By consulting the truth table we can see that this can only be true if $$ B = \top$$ and $$ C = \top$$. Therefore $$ B = C$$.
  3. Third Case: $$ A = \top$$ and $$R = \bot$$. By substituting the value of $$ A$$ we get $$ \top \oplus B = \top \oplus C = \bot$$. By consulting the truth table we can see that this can only be true if $$ B = \top$$ and $$ C = \top$$. Therefore $$ B = C$$.
  4. Fourth Case: $$ A = \top$$ and $$R = \top$$. By substituting the value of $$ A$$ we get $$ \top \oplus B = \top \oplus C = \top$$. By consulting the truth table we can see that this can only be true if $$ B = \bot$$ and $$ C = \bot$$. Therefore $$ B = C$$.

We have considered every possible case and in each case we have concluded that $$ B = C$$.

## Comparison of Proofs

So I have proved that for addition of reals $$ A + B = A + C$$ implies $$ B = C$$ and that for the exclusive or operation with Boolean values that $$ A \oplus B = A \oplus C$$ implies $$ B = C$$. It is less than elegant however that the proofs look completely different. Is it possible to identify some traits between the two scenarios and then use this information to write one proof that works for both? Indeed there is, and that is what abstract algebra allows us to do.

So what are the traits? I'll use the first example to call out some of the necessary traits of addition of reals that made the first proof work.

  1. Addition is a rule that is closed with respect to reals. This means that no matter what two real numbers we add, we end up with a result that is also a real number. We never end up with a result that is not a real. (For example, we never end up with a complex number of the form $$ a + bi$$ where $$ b \ne 0$$).
  2. In Step 2 of the proof we relied on the fact that every real number has an "opposite". In this case the opposite of a real number with respect to addition is the negative of that number. In general we call this the inverse of an element.
  3. In Step 3 of the proof we relied on the fact that addition with respect to real numbers is associative.
  4. In Step 4 of the proof we relied on the fact that when a real number is combined with it's inverse we get some known value. This known value we generally call the identity element. In this case the identity element is $$ 0$$.
  5. In Step 5 of the proof we relied on the fact that any real number added with zero (the identity element) equals the original number.

Well let's see if we can apply these assumptions to the second proposition and then see if we can re-use the structure of the original proof for the second proof. I didn't precisely define everything in the list above, so I'll just naively try to fulfill the same constraints for Booleans and exclusive or:

  1. Exclusive or is a rule that is closed with respect to Booleans. So we meet this assumption.
  2. What is the opposite of a Boolean? We might assume the opposite of $$ \bot $$ is $$ \top $$ and the opposite of $$ \top $$ is $$ \bot $$. That's what we normally think of. Right?
  3. Exclusive Or is associative. (The proof is trivial by case analysis and left to the reader.)
  4. What happens when we combine an element and it's opposite: $$ \bot \oplus \top = \top$$ and $$ \top \oplus \bot = \top$$. Therefore we can pick our identity element to be $$\top$$ since it is the result of combining opposites.
  5. Now what happens when we combine the identity element with another element. $$ \bot \oplus \top = \top$$. Oops. This isn't want we wanted. To mirror our addition example we should be able to exclusive or any Boolean with the identity element $$\top$$ and get back the original element.

So something went wrong. What is it? Well, I naively chose how to calculate the inverse of an element and also what the identity element is. It turns out that the inverse of an element depends on the operation and on the value of the identity element. The inverse of a real number with respect to addition is it's negative and the identity element is 0.

The inverse of any real number (except 0) with respect to multiplication is it's reciprocal and the identity element is $$ 1$$. We can see this by noticing that $$ A * (1/A) = A$$ for all real numbers except 0.

So let's be a little more precise. First I'll introduce some nomenclature. We use the symbol, $$ \circ$$, to stand for any operation when we aren't talking about any specific operation. The inverse of an element $$ A$$ we generically write as $$ A^{-1}$$. The identity element we generically write as $$ e$$. When choosing an inverse function and a neutral element we insist that the following holds:

- $$ e \circ A = A \circ e = A$$ (The identity element combined with another element results in the other element)

- $$ A \circ A^{-1} = A^{-1} \circ A = e$$ (The combination of an element and its inverse is always the identity element)

If you examine the truth table you'll see that the only way to fulfill these two requirements is to choose $$ e = \bot$$ and $$ A^{-1} = A$$. Let's double check.

We see that the identity element, $$ \bot $$, combines with the other elements properly

$$
\begin{align*}
\bot \oplus \bot &= \bot \\
\bot \oplus \top = \top \oplus \bot &= \top
\end{align*}
$$

and each element combined with it's inverse (itself) equals the identity element.

$$
\begin{align*}
\bot \oplus \bot^{-1} = \bot \oplus \bot = \bot \\
\top \oplus \top^{-1} = \top \oplus \top = \bot
\end{align*}
$$

Now we can redo the second proof.

$$
\begin{align*}
(A \oplus B) &= (A \oplus C) && \text{Given} \tag{1} \\
A^{-1} \oplus (A \oplus B) &= A^{-1} \oplus (A \oplus C) \tag{2} \\
(A^{-1} \oplus A) \oplus B &= (A^{-1} \oplus A) \oplus C && \text{Associativity (2)}\tag{3} \\
\bot \oplus B &= \bot \oplus C \tag{4} \\
B &= C \tag{5}
\end{align*}
$$

On line 3, remember that the inverse of $$A$$ is $$A$$ and then consult the truth table to see that $$\bot \oplus \bot = \bot$$ and $$\top \oplus \top = \bot$$. So no matter what $$A$$ is we know the result is $$\bot$$. On line 4, remember that we've already shown that $$\bot$$ combined with any element results in that element.

We have now proven the second proposition using a proof whose structure is identical to the first proof.

## Does This Proof Work for Every Set and Every Operation?

This proof only works when the constraints from the previous sections apply to the set of elements and the operation being studied. For example, it is not the case that given the set of Booleans $$ A \vee B = A \vee C$$ implies $$ B = C$$. (The symbol $$\vee$$ is the logical or operator.) Why not? It turns out that their is no inverse function that applies. While we might be safe in picking $$ \bot $$ as the identity element their is no element, $$ A$$, for which $$ A \vee \top = \bot$$.

## The Generic Proof

It turns out that this proof works for any set of elements, $$S$$, and any operation[^f2], $$\circ$$, if the following is true.

  1. The operation $$\circ$$ is associative.
  2. There is an element $$e \in S$$ such that $$a \circ e = e \circ a = a$$ for every element $$a \in S$$.
  3. Every element $$a \in S$$ has an inverse $$a^{-1} \in S$$ such that $$a \circ a^{-1} = a^{-1} \circ a = e$$.

  If all these rules are true then we call the pair $$(S,\circ)$$ a [group][7]. A group is one of the fundamental algebraic structures. I'm on Chapter 8 of the text and we are still just discussing groups. The two groups discussed in this post are $$(\mathbb{R},+$$) and $$(\mathbb{B},\oplus$$) where $$\mathbb{R}$$ is the set of real numbers and $$\mathbb{B}$$ is the set of Boolean values.

  As you might guess the generic proof looks like the following:

  $$
  \begin{align*}
  (A \circ B) &= (A \circ C) && \text{Given} \tag{1} \\
  A^{-1} \circ (A \circ B) &= A^{-1} \circ (A \circ C) \tag{2} \\
  (A^{-1} \circ A) \circ B &= (A^{-1} \circ A) \circ C && \text{Associativity (2)}\tag{3} \\
  e \circ B &= e \circ C && \text{Definition of inverse (3)} \tag{4} \\
  B &= C && \text{Definition of identity (4)} \tag{5}
  \end{align*}
  $$


 [2]: http://en.wikipedia.org/wiki/Algebraic_structure
 [3]: http://en.wikipedia.org/wiki/Exclusive_or
 [4]: http://en.wikipedia.org/wiki/Boolean_algebra
 [5]: http://en.wikipedia.org/wiki/Case_analysis
 [6]: http://en.wikipedia.org/wiki/List_of_logic_symbols
 [7]: http://en.wikipedia.org/wiki/Group_(mathematics)
 [^f1]: My text editor was informing me that I should capitalize Boolean. I Googled this and found confirmation. The odd thing is that the first hit was from Eric Lippert from [Fabulous Adventures in Coding.](http://blogs.msdn.com/b/ericlippert/archive/2006/10/31/boolean-or-or-boolean-or.aspx) It’s a blog I’ve subscribed to for years. Although I don’t ever recall this particular post. If you are a C# developer I highly recommend his blog.
 [^f2]:Note, I haven’t really defined operation. Formally, if $$A$$ is a set, then an operation $$\circ$$ on $$A$$ is a rule which assigns to each ordered pair $$(a,b)$$ of elements in $$A$$ exactly one element $$a \circ b \in A$$.
