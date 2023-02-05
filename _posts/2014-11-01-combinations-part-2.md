---
title: "Combinations - Part 2"

layout: post
permalink: /2014/11/01/combinations-part-2/
categories:
  - computer science
tags:
  - collections
  - immutable
  - swift
excerpt: ""
---

Last time I started working on the combinations problem that Eric Lippert wrote
about: [Collections](1)

We'll be working a lot with `SequenceType`, `SequenceOf`, `GeneratorType` and
`GeneratorOf`. I'm more familiar with the enumerating type of C# (`IEnumerable`
and `IEnumerator`). Together `SequenceType` and `SequenceOf` play the role of
`IEnumerable`. The other two types play the role of `IEnumerator`. I found these
types initially very confusing. `SequenceType` and `GeneratorType` are protocols
which are a little like interfaces, but you can't specify a generic type
parameter when using them. For example you can't declare our `ImmutableStack` to
implement `SequenceType<T>` (Note, I'm using parentheses rather than angle
brackets because I can't figure out how to get them to render). We can only say
it implements `SequenceType`.

Let's look at the code from last time again:

```swift
extension ImmutableStack: SequenceType {
  public func generate() -> GeneratorOf<T> {
    var stack = self

    return GeneratorOf {
      if stack.isEmpty() {
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
{: .nolineno }

How does a user of this stack know that it enumerates `T` values? Well, in
addition to the `generate` method, a `SequenceType` defines an associated type
as well via a type alias. Here is the signature of `SequenceType`

```swift
protocol SequenceType : _Sequence_Type {
  typealias Generator : GeneratorType
  func generate() -> Generator
}
```
{: .nolineno }

It says that I need to define a type alias for a type that implements
`GeneratorType`; and I must use this type as the return type of my `generate`
method. However, I still don't see any element type mentioned. Let's look at
`GeneratorType`:

```swift
protocol GeneratorType {
  typealias Element
  mutating func next() -> Element?
}
```
{: .nolineno }

Finally we see an "element" type mentioned. The `GeneratorType` protocol defines
an associated type for elements and we can see that this type is used as the
return type of the `next` method.

Now I didn't define any type aliases, so what is going on? Using inference the
compiler can often figure out what the associated type is, even if the developer
doesn't specify it. In my case the return type of my `generate` method is
`GeneratorOf<T>`, and therefore the compiler knows that the associated type for
my stack is `GeneratorOf<T>`. If you then look at the signature for
`GeneratorOf<T>` you'll see that it has a next method that returns a `T?`.

```swift
struct GeneratorOf<T> : GeneratorType, SequenceType {
    init(_ nextElement: () -> T?)
    init<G : GeneratorType where T == T>(_ base: G)
    mutating func next() -> T?
    func generate() -> GeneratorOf<T>
}
```
{: .nolineno }

I'm allowed to use `GeneratorOf<T>` as my associated type in my `ImmutableStack`
because it implements `GeneratorType`. Also the compiler infers that the `Element`
associated type for `GeneratorType` is `T`.

So putting it all together, it is now clear that my `ImmutableStack` enumerates
`T` values.

[1]: http://ericlippert.com/2014/10/13/producing-combinations-part-one/
