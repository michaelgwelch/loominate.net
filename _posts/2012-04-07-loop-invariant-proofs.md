---
title: Loop Invariant Proofs
author: Michael
layout: post
permalink: /2012/04/07/loop-invariant-proofs/
keywords:
  - brainfuck, brainmess, algorithm
categories:
  - computer science
tags:
  - algorithm
  - loop invariant
  - proof
---
Sometime last year I ran across a blog talking about loop invariant proofs. <!--more--> At the time it seemed like a new concept to me. My undergraduate degree was not in computer science, and my graduate degree in computer science was done part time over the course of many many years. Tonight I got out my graduate text on algorithms, and sure enough there on page 17 is loop invariants. I had learned it and then sometime later forgot I had ever seen it. (Even though looking at the text I can see I would have had to write loop invariant proofs for the course.)

Today, I want to use the technique of loop invariant to prove the correctness of the `FindMatch` algorithm used in [Brainmess][1].

The purpose of `FindMatch` is to find the matching bracket in a string of characters, given the index of a bracket (either left or right).

Here is the code from the previous post (with some variables renamed).

```csharp
public static class StringExtensions
{
    /// <summary>
    /// Finds the match for the bracket pointed to by
    /// pc in the program. Increment tells the algorithm
    /// which way to search.
    /// </summary>
    /// <param name="value"></param>
    /// <param name="index"></param>
    /// <param name="increment"></param>
    /// <returns></returns>
    private static int FindMatch(this string value, int index, int increment)
    {
        int nestLevel = 1;
        index += increment;
        while (nestLevel > 0)
        {
            char instruction = value[index];
            if (instruction == '[') nestLevel += increment;
            else if (instruction == ']') nestLevel -= increment;
            index += increment;
        }
        return index - increment;
    }

    public static int FindMatch(this string value, int index)
    {
        if (value[index] == '[') return value.FindMatch(index, 1);
        if (value[index] == ']') return value.FindMatch(index, -1);
        throw new ArgumentException("The character at specified location is not a square bracket");
    }
}
```

# The Loop Invariant

The loop invariant proof technique is useful when an algorithm involves looping. It requires that one identify some property about the loop that is true before the loop starts and is true after every iteration of the loop. This property is the &#8220;invariant&#8221;. Once you can prove that the property is actually invariant, then you use the property to prove the correctness of your algorithm. However, you must also prove that the algorithm terminates. So these are the three things we must show:

  1. Initialization: The property holds before executing the loop, after initialization is complete.
  2. Maintenance: The property holds after each iteration of the loop.
  3. Termination: The loop eventually terminates.

## The Algorithm

The publicly exposed `FindMatch` method is called with a string and an integer. The string (`value`) contains some series of characters and presumably some matched pairs of potentially nested brackets. The number (`index`) is the zero based index of the string that contains a bracket (either a left or a right bracket). The job of `FindMatch` is to find that brackets match and return the index value of that match.

You can see the public method is very short. It's just checking to make sure the method was called appropriately and that `index` actually points to a bracket. If it does the private method is called with the same two parameters plus a third number that is either 1 or -1 to control which way we will search the string. If the current char is a &#8216;[&#8216; then we want to search right and use an increment of 1. Otherwise, we want to search left and use an increment of -1.

The real work is done in the private `FindMatch` method. Starting with the specified index it either moves forward or backward thru the string looking for a match. It keeps track of nested brackets by maintaining a `nestLevel` variable. This variable is initialized to 1 to indicate that when we start out we are 1 level deep relative to the specified bracket character (we might actually be nested much deeper relative to the overall string, but we don't need to care about that).

As we move to the right, as we encounter &#8216;[&#8216; characters the nestLevel increases and as we encounter &#8216;]' characters the nestLevel decreases. When the nestLevel hits 0 we know we have moved past the matching &#8216;] character. Conversely, if need to move to the left, each &#8216;]' character increases the nest level and each &#8216;[&#8216; character decreases the nest level.

**The Invariant**  
The invariant in our algorithm is that the `nestLevel` variables always tells us how many levels deep we are nested within matching brackets (relative to the starting bracket). So if `nestLevel == 2` then we are nested within two levels of matching brackets.

For example, assume we have the following string

```csharp
 0123456789
"[++[>[-]]]"
```

And assume we call `FindMatch` with an index of 3. The public method will determine that we need to search right with an `increment=1` and will call the private `FindMatch`. This method will move the index to 4 and set the `nestLevel` equal to 1. This means we are 1 level deep relative to the bracket at index 3. Note if we look at the entire string we can see that at index 4 we are actually nested 2 levels deep because there is an opening bracket at position 0 and position 3. We don't care about the nesting level globally. Just relative to the starting bracket.

**Initialization**  
In the first two lines of the private `FindMatch` method we move the index 1 position (either forward or backward depending on the value of `increment`). This means that we are nested within one pair of matching brackets. (relative to the starting bracket). So we know that the invariant is true before the first iteration and after initialization is complete.

**Maintenance**  
Assuming that the `nestLevel` is correct at the beginning of an iteration we want to show that it is correct at the end of an iteration. This will tell us that we are properly maintaining the invariant.

In this algorithm we examine the character pointed to by the current index value, examine it to see what to do with the nestLevel, and then increment the index. We increment the nest level if we encounter a new `[` character and we decrement the nest level if we encounter a new `]` character. In this way we guarantee that `nestLevel` always reflects the nesting level relative to the starting bracket.

**Termination**  
Now we have to prove that the loop eventually terminates. For this algorithm it is trivial. On every iteration of the loop we modify the index by 1 (if we are searching forward) or -1 (if we are searching backward). Assuming that the brackets are matched properly then we will eventually find the match and the nestLevel will equal 0 and we terminate.

Notice I said, *if the brackets are matched*. Even if the brackets aren't matched we are guaranteed to terminate. Eventually we'll get an index out of bounds exception which will terminate the algorithm. But the point is, the algorithm will terminate and return the right result if the inputs are good. So it's important that we state a pre-condition of our method is that the brackets must be matched. It doesn't really make sense to ask for the matching bracket if we aren't even sure there is a matching bracket.

**Proof**
Ok, now I just want to put it all together. So we know that we terminate when the nestLevel is 0. And if you look at the loop you'll see that we moved the index 1 past the bracket that caused us to decrement the nestLevel to 0. So the matching bracket is at position `index - increment`.

And that's it. A simple proof that the algorithm for finding matching brackets is correct.

 [1]: http://loominate.net/2012/03/19/brainmess-extract-jump-methods/ "Brainmess: Extract Jump Methods"
