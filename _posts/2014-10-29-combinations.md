---
title: 'Combinations - Part 1'

layout: post
permalink: /2014/10/29/combinations/
categories:
  - computer science
tags:
  - collections
  - immutable
  - swift
excerpt: Follow along in the use of Swift collection types to implement graph traversal.
---
I've had a difficult time trying to understand the collections in the Swift programming language from Apple. In particular, I've been trying to identify the analog of the IEnumerable type from C#. I don't think there is one. As an exercise, I thought I'd try to port the C# code Eric Lippert wrote about Combinations. Here's a link to the start of his series: [Eric Lippert on Combinations][1]

I'm going to start with defining Stacks like he does.

Right off the bat we run into problems. Swift doesn't have abstract classes, nor does it support nested generic classes. So I did away with the abstract class and made the parent class ImmutableStack play the role of EmptyStack.

It's not that big of a deal that nested classes aren't supported. I can mark the subclass private or leave it module scope. It accomplishes the same thing as consumers of this stack will not know of its existence.

UPDATE 11/17/2014
I just was reminded that I don't need semicolons everywhere.

```swift
public class ImmutableStack<T> {

    let top:T? = nil

    private init(top:T? = nil) {
        self.top = top
    }

    public func push(t:T) -> ImmutableStack<T> {
        return NonEmptyStack(top: t, tail: self)
    }

    public func pop() -> ImmutableStack<T>? {
        return nil
    }

    public func isEmpty() -> Bool {
        return true
    }

    public class func emptyStack() -> ImmutableStack<T> {
        return ImmutableStack()
    }

}

private class NonEmptyStack<T> : ImmutableStack<T> {
    let tail:ImmutableStack<T>

    init(top:T, tail:ImmutableStack<T>) {
        self.tail = tail
        super.init(top: top)
    }

    override func push(t: T) -> ImmutableStack<T> {
        return NonEmptyStack(top: t, tail: self)
    }

    override func pop() -> ImmutableStack<T> {
        return tail
    }

    override func isEmpty() -> Bool {
        return false
    }

}
```

To make our stack `enumerable` we need to make it implement SequenceType. I've chosen to do this in the same file but took advantage of Swift extensions. I need to implement the `generate` method. Since Swift doesn't yet have a `yield` keyword, we need to manually write our generation code. There is a helper structure named `GeneratorOf`. It has a constructor that takes a closure. This closure is used to return the &#8220next&#8221 element.

```swift
extension ImmutableStack : SequenceType {
    public func generate() -> GeneratorOf<T> {
        var stack = self

        return GeneratorOf {
            if (stack.isEmpty()) {
                return nil
            } else {
                let result = stack.top
                stack = stack.pop()!
                return result
            }
        }
    }
}
```

 [1]: http://ericlippert.com/2014/10/13/producing-combinations-part-one/ "Eric Lippert on Combinations"
