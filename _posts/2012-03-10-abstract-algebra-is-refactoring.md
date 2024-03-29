---
title: Abstract Algebra is Refactoring
layout: post
permalink: /2012/03/10/abstract-algebra-is-refactoring/
categories:
  - mathematics
  - software development
tags:
  - abstract algebra
  - refactoring
math: true
---
In my previous [post][1] I presented a little introduction to abstract algebra. The easiest way for me to explain abstract algebra to a software developer is that it's a lot like refactoring.

 [1]: {{site.baseurl}}{% post_url  2012-03-07-what-is-abstract-algebra %} "What is Abstract Algebra"
<!--more-->


What do I mean by this? Well in my last post I proved two propositions that looked very similar. However, initially, the two proofs were completely different. By stepping back and looking at what the two algebraic structures had in common I was able to demonstrate one proof that applied to both propositions.

Maybe, it's just me (or maybe this is a well known concept in the world of tenured mathematicians and computer scientists) but this feels an awful lot like the process a developer uses when he's refactoring to remove duplicated code or refactoring to generalize some concept.

Let's assume that I want to print out the elements of two lists. The first list is a linked list so the psuedocode looks something like this:

```csharp
node = list.FirstNode;
while (node != null)
{
    print node.Data;
    node = node.Next;
}
```

The second list is an array based list and so the psuedocode looks something like this:

```csharp
for (int i = 0; i < list.Count; i++)
{
    print list[i]
}
```

The two examples are close enough for us to realize there is a pattern. We are visiting each element in the list one time and in some order appropriate for the list. And for each element in the list we print its value.

So just like in the abstract algebra example from last time we formally identify the common traits and "abstract" them out and give them a name. In an object orientated language this is often accomplished by defining an interface with a well chosen name. In abstract algebra we formally state that an algebraic structure has certain traits (by demonstration or perhaps by a proof). In code we formally state that a class has some well defined set of traits by implementing the interface that defines the traits.

The key things we need to be able to do for this example are the following:

  1. Create a list traversal object
  2. Using the traversal object, ask if there are any more elements
  3. If there are more elements, traverse to the next one
  4. After traversing to a new element, request it's value

In fact in C# this pattern is already formalized. The traversal of a collection is formalized by the `IEnumerator` interface which fulfills the last 3 points, and any collection which allows traversal implements the `IEnumerable` interface which fulfills the first point. The code for traversing any type of list in C# is the following[^f1].

```csharp
// Assume a list of ints
IEnumerator<int> enumerator = list.GetEnumerator();
while(enumerator.MoveNext())
{
    Console.WriteLine(enumerator.Current);
}
```

In Java we have a very similar formalization. The ability to request a list traversal object is formalized in the `Collection` interface which defines a method named `iterator` that returns an instance of an object that implements the `Iterator` interface. So the code to traverse and print a list in Java looks like the following.

```java
// Assume a list of ints
Iterator<Integer> iterator = list.iterator();
while(iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

To me, the ability to refactor two different routines into one common routine by the use of interfaces, feels very much like the process of replacing two different proofs with one common proof by the use of abstract properties of algebraic structures. Hence my claim that abstract algebra is refactoring.

In the previous post we created an abstract concept called a group. I listed the properties of a group. Then I proved a proposition that only relied on the properties of a group. I also demonstrated how both $$ (\mathbb{R},+)$$ and $$(\mathbb{B},\oplus) $$ have the properties of a group and therefore the proof applies to them.

In this post I identified some abstract concepts (interfaces) which are named `IEnumerator` and `IEnumerable` (and `Collection` and `Iterator` in Java). We then wrote routines that only relied on the methods defined by those interfaces. Since both the `LinkedList` and `List` classes of .Net implement these interfaces we know that the routine works for instances of both classes. And the same goes for the `ArrayList` and `LinkedList` classes in Java which both implement `Collection`.


 [^f1]: Yes, you can also just use the foreach syntax but behind the scenes it uses the `IEnumerable` and `IEnumerator` interfaces. Well sorta. Instead of implementing those interfaces you can also just make your objects look like they implement those interfaces. See [C# Spec](http://msdn.microsoft.com/en-us/library/aa664754(v=vs.71).aspx)
