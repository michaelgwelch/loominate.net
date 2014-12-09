---
title: 'Combinations &#8211; Part 3 (SequenceOf extensions)'
author: Michael
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
I want to continue on with my implementation of Combinations. But first, when working on this problem I determined I really needed some helper methods on SequenceOf. Initially these were very complicated but as my comfort level increased I figured out how to simplify some of them so much that they hardly provide any value anymore. Let&#8217;s take a look.

First I wanted some way to create an empty sequence. This would play the same role as Enumerable.Empty() in .Net. I couldn&#8217;t find any such built in method. I did find something analogous for generators as there exists a GeneratorOfOne structure that can be used to create GeneratorTypes that yield 0 or 1 elements. However, generators are not what I want. I want sequences. The differences are subtle but basically the difference is that everytime you call generate on a Sequence you get a &#8220;fresh&#8221; generator starting from the beginning of your sequence. This is not necessarily true of generators which are not required to &#8220;reset&#8221; back to the beginning. Anyway see [Airspeed Velocity][1] for more info. 

So here is the method:

<pre class="brush: swift; title: ; notranslate" title="">extension SequenceOf {
    public static func empty() -&gt; SequenceOf&lt;T&gt; {
        return SequenceOf([]);
    }
}
</pre>

Next, I wanted an extensions method for creating a singleton sequence.

<pre class="brush: swift; title: ; notranslate" title="">extension SequenceOf {
    public static func singleton(t:T) -&gt; SequenceOf&lt;T&gt; {
        return SequenceOf([t]);
    }
}
</pre>

As you can see this are rather simple one liners. However, I still think they are useful because they make it clear when I use them what I&#8217;m trying to do. The one liners themselves are a little more confusing. Indeed, the first one seems to say to me that I&#8217;m creating a sequence that contains one element that is an empty array. Of course, this isn&#8217;t true. Rather it&#8217;s a sequence over the elements in the array. 

Both of these methods are static methods so here is how you would invoke them:

<pre class="brush: swift; title: ; notranslate" title="">let emptySequence = SequenceOf&lt;Int&gt;.empty();
let singletonSequence = SequenceOf&lt;String&gt;.singleton("hello");
</pre>

Finally, I could not find anyway to concatenate two sequences. I looked for concat, append, join, extend methods. While these exist for arrays, collections, strings, etc. I couldn&#8217;t find anything that applied to sequences; which seems very strange to me. So I created the method. It&#8217;s fairly simple but definitely provides value.

<pre class="brush: swift; title: ; notranslate" title="">extension SequenceOf {
    public func extend&lt;S1:SequenceType where S1.Generator.Element == T&gt;(s1:S1) -&gt; SequenceOf&lt;T&gt; {
        return SequenceOf{ () -&gt; GeneratorOf&lt;T&gt; in
            var g0 = self.generate();
            var g1 = s1.generate();
            return GeneratorOf {
                g0.next() ?? g1.next();
            }
        }

    }
}
</pre>

This might take a little explanation for beginners. 

First, let&#8217;s examine the signature. It&#8217;s a generic method that takes one type parameter `S1`. Their are constraints placed on what S1 can be. First of all it must implement the SequenceType protocol. Next there is a *where* clause that states that the Element type of S1 must be identical to T. What is T? Well it helps to recall that SequenceOf is a generic struct. T is the type parameter for SequenceOf. So what the constraints on S1 are saying is that S1 can be any SequenceType that has the same type of elements as the sequence this method is being called on. So we could.

Next we see the arguments for this method. It takes one, s1 of type S1. Finally, the return type is SequenceOf<T>. This makes sense because it&#8217;s the most fundamental SequenceType (except for maybe Array).

The rest of the method is a little complicated so I&#8217;m going to rewrite the method and be explicit:

<pre class="brush: swift; title: ; notranslate" title="">func extend&lt;S1:SequenceType where S1.Generator.Element == T&gt;(s1:S1) -&gt; SequenceOf&lt;T&gt; {

    return SequenceOf&lt;T&gt;({ () -&gt; GeneratorOf&lt;T&gt; in
        var g0 = self.generate();
        var g1 = s1.generate();
        return GeneratorOf&lt;T&gt;({ () -&gt; T? in
            g0.next() ?? g1.next();
        })
    })

}
</pre>

I&#8217;m instantiating and returning a SequenceOf. It has a constructor that takes a closure. The closure it is expecting is one that takes no parameters and returns a GeneratorType. The implementation of this closure first calls generate on self and s1 and stores those generators in vars. (They must be vars since the next method that will be called on them is mutating.) Then I instantiate and return a GeneratorOf. Now GeneratorOf also has a constructor that takes a closure. This closure is basically the &#8220;next&#8221; method you want the GeneratorOf to have. Our next method is pretty simple. It calls next on the first generator and if the result is nil then it calls next on the second generator.

NB: It&#8217;s very important where I instantiated g0 and g1. If I would have instantiated them outside of the closure, (for example if they were the first two lines in the extend method); then the generate methods would only be called when the extend method was called. This means that the resulting sequence wouldn&#8217;t reset when it&#8217;s generate method is called. 

If I would have called generate on g0 and g1 inside the nested closure then the sequence would reset overtime next is called and always returns the first element of g0. Let me illustrate what I&#8217;m saying by actually making these mistakes and then showing the results.

<pre class="brush: swift; title: ; notranslate" title="">// Error #1. Sequence doesn't "reset" after it's been enumerated
extension SequenceOf {
    func extend3&lt;S1:SequenceType where S1.Generator.Element == T&gt;(s1:S1) -&gt; SequenceOf&lt;T&gt; {

        var g0 = self.generate();
        var g1 = s1.generate();

        return SequenceOf&lt;T&gt;({ () -&gt; GeneratorOf&lt;T&gt; in
            return GeneratorOf&lt;T&gt;({ () -&gt; T? in
                return g0.next() ?? g1.next();
            })
        })
        
    }
}

var items = SequenceOf([1,2]).extend3([3]);
var gen = items.generate();
println(gen.next());
println(gen.next());
println(gen.next());
println(gen.next());
gen = items.generate();
println(gen.next());
println(gen.next());
println(gen.next());
println(gen.next());
</pre>

If you copy/paste this into a playground you&#8217;ll see the output is 

<pre class="brush: plain; title: ; notranslate" title="">Optional(1)
Optional(2)
Optional(3)
nil
nil
nil
nil
nil
</pre>

In other words you can see that once the sequence is exhausted it wasn&#8217;t reset even after calling generate method again to get a fresh generator.

Now let&#8217;s try the other mistake

<pre class="brush: swift; title: ; notranslate" title="">// Error #2, we never make any progress
extension SequenceOf {
    func extend4&lt;S1:SequenceType where S1.Generator.Element == T&gt;(s1:S1) -&gt; SequenceOf&lt;T&gt; {

        return SequenceOf&lt;T&gt;({ () -&gt; GeneratorOf&lt;T&gt; in
            return GeneratorOf&lt;T&gt;({ () -&gt; T? in
                var g0 = self.generate();
                var g1 = s1.generate();
                return g0.next() ?? g1.next();
            })
        })
        
    }
}

var items = SequenceOf([1,2]).extend4([3]);
var gen = items.generate();
println(gen.next());
println(gen.next());
println(gen.next());
println(gen.next());
</pre>

You&#8217;ll see the output is

<pre class="brush: plain; title: ; notranslate" title="">Optional(1)
Optional(1)
Optional(1)
Optional(1)
</pre>

Of course this makes sense because we reset the generator on the first sequence every time next is called so we never make any progress.

Now let&#8217;s look at the original method:

<pre class="brush: plain; title: ; notranslate" title="">Optional(1)
Optional(2)
Optional(3)
nil
Optional(1)
Optional(2)
Optional(3)
nil
</pre>

Ok, this post got long enough. Let&#8217;s quit here.

 [1]: http://airspeedvelocity.net/2014/07/28/collection-and-sequence-helpers/