---
title: "Combinations - Part 4 (combinations of booleans)"

layout: post
permalink: /2014/11/01/combinations-part-4-combinations-of-booleans/
categories:
  - computer science
tags:
  - algorithms
  - collections
  - immutable
  - swift
---

In this article we'll implement the `combinations` function from Eric Lippert's
post. Here he implements a function that returns all the combinations of `n`
booleans with `k` of them set to true.

<!--more-->

See [part 3][1] of his series.

Let's implement in parts like Eric did, starting with some preliminaries and the
signature

```swift
typealias BoolStack = ImmutableStack<Bool>;
let singletonEmptyBoolStack = SequenceOf.singleton(BoolStack.emptyStack());
let emptySequenceOfBoolStack:SequenceOf<BoolStack> = SequenceOf.empty();

func combinations(n:UInt, k:UInt) -> SequenceOf<BoolStack> { }
```
{: .nolineno }

I'm going to use a type alias to reduce slightly the amount of typing I need to
do to get an `ImmutableStack<Bool>`. Next I'm going to define some variables for
sequences that will be used in base cases of our recursion. The first is a
singleton sequence that contains one empty boolean stack. The second is a
completely empty sequence.

I changed the signature slightly to take UInts rather than Ints. This just
reduces one error case I need to check for. We are going to return a sequence of
boolean stacks.

I'm going to follow the exact same algorithm, so I have the first two simple
base cases.

```swift
if (k == 0 && n == 0) {
    return singletonEmptyBoolStack;
}

if (n < k) {
    return emptySequenceOfBoolStack;
}
```
{: .nolineno }

Note that we don't have a yield statement so we need to return a "full
sequence".

Now the base cases are done. Like Eric we have two cases left to handle. The
first are the cases where we are going to enumerate the combinations(n-1,k-1)
and push a true on to them.

```swift
let seq1 = (k>0)
        ? SequenceOf(lazy(combinations(n-1, k-1)).map({$0.push(true)}))
        : emptySequenceOfBoolStack;
```
{: .nolineno }

If k is greater than 0 then we can call combinations(n-1,k-1). If it isn't then
seq1 will be empty. Now for every combination returned by combinations(n-1,k-1)
we want to push a true onto it. We can do this with a map function rather than
iterating thru all of the combinations. Strangely map isn't defined for regular
SequenceOf instances. It is only defined for LazySequence instances. We can
easily create a LazySequence from any SequenceType by calling the lazy method.
However, then when we are done mapping we no longer have a SequenceOf. (map
returns a LazySequence as well). But we can easily convert back to a SequenceOf
as SequenceOf has a constructor that takes any other SequenceType.

So this highlights some of the strangeness of the library currently. Sequences
are kind of clumsy to use and require lots of conversions like in this example.
If you want to avoid the two conversions you can define your own map method for
any SequenceType. I did it like this:

```swift
extension SequenceOf {
    func map<U>(transform:T -> U) -> SequenceOf<U> {

        return SequenceOf<U> { () -> GeneratorOf<U> in
            var g = self.generate();
            return GeneratorOf {
                return g.next().map(transform);
            }
        }
    }
}
```
{: .nolineno }

This would the simplify our code above to look like this (which you have to
admit looks much nicer, but I was trying to figure out how to use the built-in
libraries so I tried both ways)

```swift
let seq1 = (k>0) ? combinations(n-1,k-1).map({$0.push(true)}) : emptySequenceOfBoolStack;
```

Next we need to generate the second sequence which is combinations(n-1,k) where
we push a false onto each combination. And then we want to combine the two
sequences:

```swift
return seq1.extend(
    SequenceOf(lazy(combinations(n-1, k)).map({$0.push(false)})));

// again this could be simplified with my custom map function to this
// return seq1.extend(combinations(n-1, k).map({$0.push(false)}));
```
{: .nolineno }

Again, note that are code is a little more difficult because we can't take
advantage of yield, rather I'm creating complete sequences. But also note that
the definition of my map method doesn't really eagerly evaluate the sequences,
nor does my extend method. So we are still returning lazy lists.

That's it for this function.

Here is is with the built-in lazy functions

```swift
func combinations(n:UInt, k:UInt) -> SequenceOf<BoolStack> {
    if (k == 0 && n == 0) {
        return singletonEmptyBoolStack;
    }

    if (n < k) {
        return emptySequenceOfBoolStack;
    }

    let seq1 = (k>0)
        ? SequenceOf(lazy(combinations(n-1, k-1)).map({$0.push(true)}))
        : emptySequenceOfBoolStack;
    return seq1.extend(
        SequenceOf(lazy(combinations(n-1, k)).map({$0.push(false)})));
}
```
{: .nolineno }

And here with my custom map function:

```swift
func combinations(n:UInt, k:UInt) -> SequenceOf<BoolStack> {
    if (k == 0 && n == 0) {
        return singletonEmptyBoolStack;
    }

    if (n < k) {
        return emptySequenceOfBoolStack;
    }

    let seq1 = (k>0) ? combinations(n-1,k-1).map({$0.push(true)}) : emptySequenceOfBoolStack;

    return seq1.extend(combinations(n-1, k).map({$0.push(false)}));
}
```
{: .nolineno }

[1]: http://ericlippert.com/2014/10/20/producing-combinations-part-three/
