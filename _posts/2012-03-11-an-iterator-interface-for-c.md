---
title: 'An Iterator Interface for C#'
author: Michael
layout: post
permalink: /2012/03/11/an-iterator-interface-for-c/
categories:
  - software development
tags:
  - patterns
---
The `IEnumerator` interface of C# is usually fine for visiting a collection of elements. However, often times when I&#8217;m doing something more complicated I want to split up the functionality of `MoveNext`<sup class='footnote'><a href='http://loominate.net/2012/03/11/an-iterator-interface-for-c/#fn-156-1' id='fnref-156-1' onclick='return fdfootnote_show(156)'>1</a></sup>. In this post I&#8217;ll define a new interface and show how to employ the Adapter pattern to implement it.

<!--more-->

`MoveNext` does two things: 1) it moves the cursor forward one position and 2) returns a `bool` value that indicates if there is an element at the current position. There are occasions where I want that boolean persisted. This requires me to keep a local variable (or a member variable) to maintain this piece of information. I&#8217;ve done this enough times that I decided that the iterator should store this for me. So I defined the interface `IIterator`.

<noscript>
  <pre><code class="language-c# c#">using System;

namespace CollectionsExtensions
{
    /// <summary>
    /// Defines methods for iteratating over a collection.
    /// </summary>
    /// <remarks>
    /// Sometimes it's advantageous to have a HasCurrent property. This 
    /// interface differs from <see cref="IEnumerator{T}>"/> in that
    /// <see cref="MoveNext"/> has no return value, and the value it would
    /// have returned is accessible from <see cref="HasCurrent"/>
    /// property.
    /// </remarks>
    public interface IIterator<out T> : IDisposable
    {
        bool HasCurrent { get; }
        T Current { get; }
        void MoveNext();
    }
}</code></pre>
</noscript>

The `MoveNext` method now has no return value. It simply moves to the next element. You must separately inspect `HasCurrent` property to see if there is an element at the current posistion.

None of the collection classes implement this interface. However, it it trivial to write an adapter class that implements <IIterator> and can wrap an `IEnumerator`.

<noscript>
  <pre><code class="language-c# c#">using System.Collections.Generic;

namespace CollectionsExtensions
{
    /// <summary>
    /// Defines an implementation of <see cref="IIterator{T}"/> that
    /// works on any <see cref="IEnumerable{T}"/> instance by wrapping
    /// its enumerator.
    /// </summary>
    public class EnumeratorIterator<T> : IIterator<T>
    {
        private readonly IEnumerator<T> enumerator;
        private bool hasCurrent;

        public bool HasCurrent
        {
            get { return hasCurrent; }
        }

        public T Current
        {
            get { return enumerator.Current; }
        }

        public void MoveNext()
        {
            hasCurrent = enumerator.MoveNext();
        }

        private EnumeratorIterator(IEnumerator<T> enumerator)
        {
            this.enumerator = enumerator;
            this.hasCurrent = enumerator.MoveNext();
        }

        /// <summary>
        /// Gets an iterator for the specified enumerable.
        /// </summary>
        /// <param name="enumerable"></param>
        /// <returns></returns>
        public static IIterator<T> GetIterator(IEnumerable<T> enumerable)
        {
            return new EnumeratorIterator<T>(enumerable.GetEnumerator());
        }

        public void Dispose()
        {
            enumerator.Dispose();
        }
    }
}</code></pre>
</noscript>

The constructor is hidden because otherwise we could be given an enumerator that is not at the beginning of the collection. To ensure we get a &#8220;fresh&#8221; enumerator I only provide a GetIterator method which takes an `IEnumerable` from which I can request a new `IEnumerator`.

This already allows us to retrieve an iterator but we can provide a little syntactic sugar by use of an extension method.

<noscript>
  <pre><code class="language-c# c#">using System.Collections.Generic;

namespace CollectionsExtensions
{
    public static class EnumerableExtensions
    {
        /// <summary>
        /// Provides an extension method that gets an iterator for
        /// IEnumerables. This makes it as easy to get an iterator as it 
        /// is to get an enumerator.
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="enumerable"></param>
        /// <returns></returns>
        public static IIterator<T> GetIterator<T>(this IEnumerable<T> enumerable)
        {
            return EnumeratorIterator<T>.GetIterator(enumerable);
        }
    }
}</code></pre>
</noscript>

And now we can request an iterator for an enumerable object as shown in the next trivial example.

<noscript>
  <pre><code class="language-c# c#">using System;
using System.Linq;

namespace CollectionsExtensions
{
    public class Program
    {
        public static void Main()
        {
            var array = new [] { 1, 3, 5, 7, 11, 13, 17, 19 }; 
            
            // ridiculous example - since this could be simplified
            // by using foreach and an enumerator.
            var iterator = array.GetIterator();

            while (iterator.HasCurrent)
            {
                Console.WriteLine(iterator.Current);
                iterator.MoveNext();
            }


            iterator = Enumerable.Empty<int>().GetIterator();
            while (iterator.HasCurrent)
            {
                Console.WriteLine(iterator.Current);
                iterator.MoveNext();
            }

        }
    }
}</code></pre>
</noscript>
