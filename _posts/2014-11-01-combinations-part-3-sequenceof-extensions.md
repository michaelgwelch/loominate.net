---
title: "Combinations - Part 3 (SequenceOf extensions)"

layout: post
permalink: /2014/11/01/combinations-part-3-sequenceof-extensions/
categories:
  - Apple
  - computer science
tags:
  - collections
  - immutable
  - swift
---

I want to continue on with my implementation of Combinations. But first, when
working on this problem I determined I really needed some helper methods on
SequenceOf. <!--more--> Initially these were very complicated but as my comfort
level increased I figured out how to simplify some of them so much that they
hardly provide any value anymore. Let's take a look.

First I wanted some way to create an empty sequence. This would play the same
role as `Enumerable.Empty()` in .Net. I couldn't find any such built in method. I
did find something analogous for generators as there exists a `GeneratorOfOne`
structure that can be used to create `GeneratorType`s that yield 0 or 1 elements.
However, generators are not what I want. I want sequences. The differences are
subtle but basically the difference is that everytime you call generate on a
`Sequence` you get a "fresh" generator starting from the beginning of your
sequence. This is not necessarily true of generators which are not required to
"reset" back to the beginning. Anyway see [Airspeed Velocity][1] for more info.

So here is the method:

```swift
extension SequenceOf {
  public static func empty() -> SequenceOf<T> {
    return SequenceOf([]);
  }
}
```
{: .nolineno }

Next, I wanted an extension method for creating a singleton sequence.

```swift
extension SequenceOf {
  public static func singleton(t:T) -> SequenceOf<T> {
    return SequenceOf([t]);
  }
}
```
{: .nolineno }

As you can see these are rather simple one liners. However, I still think they
are useful because they make it clear when I use them what I'm trying to do. The
one liners themselves are a little more confusing. Indeed, the first one seems
to say to me that I'm creating a sequence that contains one element that is an
empty array. Of course, this isn't true. Rather it's a sequence over the
elements in the array.

Both of these methods are static methods so here is how you would invoke them:

```swift
let emptySequence = SequenceOf<Int>.empty();
let singletonSequence = SequenceOf<String>.singleton("hello");
```
{: .nolineno }

Finally, I could not find anyway to concatenate two sequences. I looked for
`concat`, `append`, `join`, `extend` methods. While these exist for `arrays`, `collections`,
`strings`, etc. I couldn't find anything that applied to sequences; which seems
very strange to me. So I created the method. It's fairly simple but definitely
provides value.

```swift
public extension SequenceOf {
  func extend<S1: SequenceType where S1.Generator.Element == T>(s1: S1) -> SequenceOf<T> {
    return SequenceOf { () -> GeneratorOf<T> in
      var g0 = self.generate()
      var g1 = s1.generate()
      return GeneratorOf {
        g0.next() ?? g1.next()
      }
    }
  }
}
```
{: .nolineno }

This might take a little explanation for beginners.

First, let's examine the signature. It's a generic method that takes one type
parameter `S1`. There are constraints placed on what `S1` can be. First of all it
must implement the `SequenceType` protocol. Next there is a _where_ clause that
states that the `Element` type of `S1` must be identical to `T`. What is `T`? Well it
helps to recall that `SequenceOf` is a generic struct. `T` is the type parameter for
`SequenceOf`. So what the constraints on `S1` are saying is that `S1` can be any
`SequenceType` that has the same type of elements as the sequence this method is
being called on.

Next we see the arguments for this method. It takes one, `s1` of type `S1`. Finally,
the return type is `SequenceOf<T>`. This makes sense because it's the most
fundamental `SequenceType` (except for maybe `Array`).

The rest of the method is a little complicated so I'm going to rewrite the
method and be explicit in all the types:

```swift
func extend<S1: SequenceType where S1.Generator.Element == T>(s1: S1) -> SequenceOf<T> {
  return SequenceOf<T>({ () -> GeneratorOf<T> in
    var g0 = self.generate()
    var g1 = s1.generate()
    return GeneratorOf<T>({ () -> T? in
      g0.next() ?? g1.next()
    })
  })
}

```
{: .nolineno }

I'm instantiating and returning a `SequenceOf`. It has a constructor that takes a
closure. The closure it is expecting is one that takes no parameters and returns
a `GeneratorType`. The implementation of this closure first calls `generate` on `self`
and `s1` and stores those generators in vars. (They must be vars since the next
method that will be called on them is mutating.) Then I instantiate and return a
`GeneratorOf`. Now `GeneratorOf` also has a constructor that takes a closure. This
closure is basically the `next` method you want the `GeneratorOf` to have. Our
next method is pretty simple. It calls next on the first generator and if the
result is `nil` then it calls next on the second generator.

NB: It's very important where I instantiated `g0` and `g1`. If I would have
instantiated them outside of the closure, (for example if they were the first
two lines in the `extend` method); then the generate methods would only be called
when the extend method was called. This means that the resulting sequence
wouldn't reset when it's generate method is called.

If I would have called generate on `g0` and `g1` inside the nested closure then the
sequence would reset every time `next` is called and always return the first
element of `g0`. Let me illustrate what I'm saying by actually making these
mistakes and then showing the results.

```swift
// Error #1. Sequence doesn't "reset" after it's been enumerated
extension SequenceOf {
  func extend3<S1: SequenceType where S1.Generator.Element == T>(s1: S1) -> SequenceOf<T> {
    var g0 = generate()
    var g1 = s1.generate()

    return SequenceOf<T>({ () -> GeneratorOf<T> in
      GeneratorOf<T>({ () -> T? in
        g0.next() ?? g1.next()
      })
    })
  }
}

var items = SequenceOf([1, 2]).extend3([3])
var gen = items.generate()
println(gen.next())
println(gen.next())
println(gen.next())
println(gen.next())
gen = items.generate()
println(gen.next())
println(gen.next())
println(gen.next())
println(gen.next())
```
{: .nolineno }

If you copy/paste this into a playground you'll see the output is

```swift
Optional(1)
Optional(2)
Optional(3)
nil
nil
nil
nil
nil
```
{: file="playground" .nolineno }

In other words you can see that once the sequence is exhausted it wasn't reset
even after calling the `generate` method again to get a fresh generator.

Now let's try the other mistake

```swift
// Error #2, we never make any progress
extension SequenceOf {
  func extend4<S1: SequenceType where S1.Generator.Element == T>(s1: S1) -> SequenceOf<T> {
    return SequenceOf<T>({ () -> GeneratorOf<T> in
      GeneratorOf<T>({ () -> T? in
        var g0 = self.generate()
        var g1 = s1.generate()
        return g0.next() ?? g1.next()
      })
    })
  }
}

var items = SequenceOf([1, 2]).extend4([3])
var gen = items.generate()
println(gen.next())
println(gen.next())
println(gen.next())
println(gen.next())
```

You'll see the output is

```swift
Optional(1)
Optional(1)
Optional(1)
Optional(1)
```
{: file="playground" .nolineno }

Of course this makes sense because we reset the generator on the first sequence
every time `next` is called so we never make any progress.

Now let's look at the results of the correct method:

```swift
Optional(1)
Optional(2)
Optional(3)
nil
Optional(1)
Optional(2)
Optional(3)
nil
```
{: .nolineno }

Ok, this post got long enough. Let's quit here.

[1]: http://airspeedvelocity.net/2014/07/28/collection-and-sequence-helpers/
