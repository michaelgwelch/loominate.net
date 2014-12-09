---
title: 'Combinations &#8211; Part 1'
author: Michael
layout: post
permalink: /2014/10/29/combinations/
categories:
  - computer science
tags:
  - collections
  - immutable
  - swift
---
I&#8217;ve had a difficult time trying to understand the collections in the Swift programming language from Apple. In particular, I&#8217;ve been trying to identify the analog of the IEnumerable type from C#. I don&#8217;t think there is one. As an exercise, I thought I&#8217;d try to port the C# code Eric Lippert wrote about Combinations. Here&#8217;s a link to the start of his series: [Eric Lippert on Combinations][1]

I&#8217;m going to start with defining Stacks like he does.

Right off the bat we run into problems. Swift doesn&#8217;t have abstract classes, nor does it support nested generic classes. So I did away with the abstract class and made the parent class ImmutableStack play the role of EmptyStack. 

It&#8217;s not that big of a deal that nested classes aren&#8217;t supported. I can mark the subclass private or leave it module scope. It accomplishes the same thing as consumers of this stack will not know of its existence.

UPDATE 11/17/2014  
I just was reminded that I don&#8217;t need semicolons everywhere. 

<pre class="brush: swift; title: ; notranslate" title="">import Foundation


public class ImmutableStack&lt;T&gt; {

    let top:T? = nil;

    private init(top:T? = nil) {
        self.top = top;
    }

    public func push(t:T) -&gt; ImmutableStack&lt;T&gt; {
        return NonEmptyStack(top: t, tail: self);
    }

    public func pop() -&gt; ImmutableStack&lt;T&gt;? {
        return nil;
    }

    public func isEmpty() -&gt; Bool {
        return true;
    }

    public class func emptyStack() -&gt; ImmutableStack&lt;T&gt; {
        return ImmutableStack();
    }

}

private class NonEmptyStack&lt;T&gt; : ImmutableStack&lt;T&gt; {
    let tail:ImmutableStack&lt;T&gt;;

    init(top:T, tail:ImmutableStack&lt;T&gt;) {
        self.tail = tail;
        super.init(top: top);
    }

    override func push(t: T) -&gt; ImmutableStack&lt;T&gt; {
        return NonEmptyStack(top: t, tail: self);
    }

    override func pop() -&gt; ImmutableStack&lt;T&gt; {
        return tail;
    }

    override func isEmpty() -&gt; Bool {
        return false;
    }

}

</pre>

To make our stack &#8220;enumerable&#8221; we need to make it implement SequenceType. I&#8217;ve chosen to do this in the same file but took advantage of Swift extensions. I need to implement the `generate` method. Since Swift doesn&#8217;t yet have a `yield` keyword, we need to manually write our generation code. There is a helper structure named `GeneratorOf`. It has a constructor that takes a closure. This closure is used to return the &#8220;next&#8221; element.

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

 [1]: http://ericlippert.com/2014/10/13/producing-combinations-part-one/ "Eric Lippert on Combinations"