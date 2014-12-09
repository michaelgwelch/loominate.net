---
title: 'Combinations Part 5 &#8211; (wrapping it up)'
author: Michael
layout: post
permalink: /2014/11/01/combinations-part-5-wrapping-it-up/
categories:
  - computer science
tags:
  - algorithms
  - immutable
  - swift
---
The original problem as posed by Eric Lippert was to write a function to produce all the combinations of k elements from a collection of n elements. So far I&#8217;ve ported the code to product all the combinations of k true booleans in a collection of n booleans where k <= n.

We can use the work we've done so far to solve the original problem. See [Lippert Part 3][1] to follow along with his post.

My complete solution is stored as a [gist here][2].

First I&#8217;ll create a zipWhere method like he did.

<pre class="brush: swift; title: ; notranslate" title="">extension SequenceOf {
    func zipWhere&lt;S:SequenceType where S.Generator.Element == Bool&gt;(bools:S) -&gt; SequenceOf&lt;T&gt; {

        return SequenceOf { () -&gt; GeneratorOf&lt;T&gt; in
            var generator = Zip2(self, bools).generate();
            return GeneratorOf&lt;T&gt; {
                var next = generator.next();

                while (next != nil) {
                    let (v,b) = next!;
                    if (b) {
                        return v;
                    } else {
                        next = generator.next();
                    }
                }
                
                return nil;
            }
        }
    }
}
</pre>

On the first line I use the Zip2 structure to create a sequence of tuples out the elements in self and the sequence of bools.

Then I return a new SequenceOf that only returns the elements from self that are paired up in a tuple with a true value from the booleans.

Now we can write the final function. It relies on my custom map function mentioned in last post. First we generate all the boolean combinations using the combinations function from last time. Then we use the map function to transform each boolean combination into a combination of the elements in s.

<pre class="brush: swift; title: ; notranslate" title="">func combinations&lt;S:SequenceType&gt;(s:S,k:UInt) -&gt; SequenceOf&lt;SequenceOf&lt;S.Generator.Element&gt;&gt; {
    let sArray = Array(s);
    let sseq = SequenceOf(s);

    return combinations(UInt(sArray.count), k).map { sseq.zipWhere($0) };
}
</pre>

See all code

 [1]: http://ericlippert.com/2014/10/20/producing-combinations-part-three/ "Lippert Part 3"
 [2]: https://gist.github.com/michaelgwelch/989b555de95b4dd06962