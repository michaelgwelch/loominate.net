---
title: 'Combinations &#8211; Part 2'
author: Michael
layout: post
permalink: /2014/11/01/combinations-part-2/
categories:
  - computer science
tags:
  - collections
  - immutable
  - swift
---
Last time I started working on the combinations problem that Eric Lippert wrote about: [][1]

We&#8217;ll be working a lot with `SequenceType`, `SequenceOf`, `GeneratorType` and `GeneratorOf`. I&#8217;m more familiar with the enumerating type of C# (`IEnumerable` and `IEnumerator`). Together `SequenceType` and `SequenceOf` play the role of `IEnumerable`. The other two types play the role of `IEnumerator`. I found these types initially very confusing. `SequenceType` and `GeneratorType` are protocols which are a little like interfaces, but you can&#8217;t specify a generic type parameter when using them. For example you can&#8217;t declare our `ImmutableStack` to implement `SequenceType(T)` (Note, I&#8217;m using parentheses rather than angle brackets because I can&#8217;t figure out how to get them to render). We can only say it implements `SequenceType`.

Let&#8217;s look at the code from last time again:

<pre class="brush: swift; title: ; notranslate" title="">extension ImmutableStack : SequenceType {
    public func generate() -&gt; GeneratorOf&lt;T&gt; {
        var stack = self;

        return GeneratorOf {
            if (stack.isEmpty()) {
                return nil;
            } else {
                let result = stack.top;
                stack = stack.pop()!;
                return result;
            }
        };
    }
}
</pre>

How does a user of this stack know that it enumerates `T` values? Well, in addition to the `generate` method, a `SequenceType` defines an associated type as well via a type alias. Here is the signature of `SequenceType`

<pre class="brush: swift; title: ; notranslate" title="">protocol SequenceType : _Sequence_Type {
    typealias Generator : GeneratorType
    func generate() -&gt; Generator
}
</pre>

It says that I need to define a type alias for a type that implements GeneratorType; and I must use this type as the return type of my generate method. However, I still don&#8217;t see any element type mentioned. Let&#8217;s look at `GeneratorType`:

<pre class="brush: swift; title: ; notranslate" title="">protocol GeneratorType {
    typealias Element
    mutating func next() -&gt; Element?
}
</pre>

Finally we see an &#8220;element&#8221; type mentioned. The `GeneratorType` protocol defines an associated type for elements and we can see that this type is used as the return type of the `next` method.

Now I didn&#8217;t define any type aliases, so what is going on? Using inference the compiler can often figure out what the associated type is, even if the developer doesn&#8217;t specify it. In my case the return type of my `generate` method is `GeneratorOf(T)`, and therefore the compiler knows that the associated type for my stack is `GeneratorOf(T)`. If you then look at the signature for `GeneratorOf(T)` you&#8217;ll see that it has a next method that returns a `T?`. 

<pre class="brush: swift; title: ; notranslate" title="">struct GeneratorOf&lt;T&gt; : GeneratorType, SequenceType {
    init(_ nextElement: () -&gt; T?)
    init&lt;G : GeneratorType where T == T&gt;(_ base: G)
    mutating func next() -&gt; T?
    func generate() -&gt; GeneratorOf&lt;T&gt;
}
</pre>

I&#8217;m allowed to use `GeneratorOf(T)` as my associated type in my ImmutableStack because it implements GeneratorType. Also the compiler infers that the Element associated type for GeneratorType is T.

So putting it all together, it is now clear that my `ImmutableStack` enumerates `T` values.

 [1]: http://ericlippert.com/2014/10/13/producing-combinations-part-one/ "Collections"