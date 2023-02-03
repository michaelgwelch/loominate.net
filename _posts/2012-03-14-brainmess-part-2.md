---
title: 'Brainmess: Part 2'
layout: post
permalink: /2012/03/14/brainmess-part-2/
keywords:
  - brainmess, brainfuck, refactoring, testing
categories:
  - software development
tags:
  - brainmess
  - refactoring
  - testing
---
In my last [post][1] I introduced you to Brainmess and presented an "all-in-one" implementation of an interpreter for it. In this post I'm going to begin the process of refactoring to address some of the concerns I had in the initial implementation.

[1]: {{site.baseurl}}{% post_url  2012-03-13-brainmess %} "Brainmess"

<!--more-->

## First Things First! Get Some Tests In Place

Last time I stated that we couldn't write any automated tests for this program. I lied. We can't write any automated unit tests, but we can write some automated tests. I'm going to write some automated tests so that while I'm doing my refactoring, I can always quickly tell if I broke something. (Note: since we don't have a full suite of tests, unit or otherwise, I won't know for sure that I didn't break something during my refactoring. However, having some tests is better than none. After all, I'm not really sure that there aren't any defects in the current program either.)

As I mentioned last time I have a handful of Brainmess scripts that I manually run to check to see if the interpreter is working. I'm going to use the NUnit framework to allow me to write some automated tests. (You could also write some bash scripts or "DOS" scripts as well.) What allows me to use NUnit is the fact that the `System.Console` class has methods that allows me to redirect standard input and standard output.

So here is my plan for my automated tests. In each test that just does output, I'm going to
redirect the output to a `TextWriter` that writes to a `StringBuilder`. Then when the program is done running (and I know what the program is supposed to do), I assert that the string produced by the `StringBuilder` matches what was expected.

For programs that also require input, I'll use a similar trick to specify the data that should be passed into standard input. Then I can assert what the output should be based on the input.

### Redirecting Output

I can redirect the standard output with the following method that redirects the output to a `StringBuilder` and returns that `StringBuilder` to the caller. On line 3 we create the `StringBuilder`. On line 4 we wrap that in a `TextWriter` which is what the `SetOut` method on line 5 is expecting. After redirecting the output, we return the builder so that the test case can use it for its asserts.

```csharp
private static StringBuilder RedirectOutput()
{
    var builder = new StringBuilder();
    var writer = new StringWriter(builder);
    Console.SetOut(writer);
    return builder;
}
```

### Redirecting Input

I can redirect the standard input with the following method that allows me to pass a string
in. That string will need to contain all of the characters necessary for the successful completion of whatever program I'm running. I then wrap that string in a `StringReader` (line 3) which I can then pass to the `SetIn` method on line 4.

```csharp
private static void SetInput(string input)
{
    var reader = new StringReader(input);
    Console.SetIn(reader);
}
```

### Test Cases

I have three scripts that I'm going to use for testing. The first script is `hello.bm` which is a _Hello World_ program. The second script is `fibonacci.bm` and it prints out the fibonacci sequence up to 89. Finally, I have a script called `double.bm` which knows how to read in one character, treat it as a number, double it, and then output the result. So for example if the input was the character `'2'`, the program would read it in, convert it to the number `2`, double it, convert the result back to a character (in this case `4`) and then output the result.

Here is the Hello World test case (Note: I have an explicit dependency on the location of the script. I could remove this by embedding the program right in the test case. This is the case for each of my tests.)

```csharp
[Test]
public void RunHelloWorld()
{
    var builder = RedirectOutput();
    Brainmess.Main(new[] {"../../../../scripts/hello.bm"});
    Assert.AreEqual("Hello World!", builder.ToString());
}
```

On line 4, I redirect the output and retrieve the `StringBuilder`. Then I run the interpreter by calling the `Main` method. As the program runs it writes its output to standard output which ends up in my `StringBuilder`. Then on line 6 I can convert the builder to a string and compare it to the expected value.

Here is the fibonacci test case. It follows the same pattern.

```csharp
[Test]
public void RunFibonacci()
{
    var builder = RedirectOutput();
    Brainmess.Main(new[] {"../../../../scripts/fibonacci.bm"});
    Assert.AreEqual("1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89",
        builder.ToString());
}
```

Here are the test cases for `double.bm`. The new wrinkle is the use of the `SetIn` method. On line 5 (in the first case) I set the input to contain the string `"0"`. When the program is run, it reads in a single character doubles it and outputs the result. The second case asserts that the program reads in a `"2"` and writes out a `"4"`.

```csharp
[Test]
public void RunDoubleWith0Expect0()
{
    var builder = RedirectOutput();
    SetInput("0");
    Brainmess.Main(new[] { "../../../../scripts/double.bm" });
    Assert.AreEqual("0", builder.ToString());
}

[Test]
public void RunDoubleWith2Expect4()
{
    var builder = RedirectOutput();
    SetInput("2");
    Brainmess.Main(new[] { "../../../../scripts/double.bm" });
    Assert.AreEqual("4", builder.ToString());
}
```

Now that we have at least some tests in place we can begin refactoring. In the next post, I'm going to address the while loops in the switch statement for the `[` and `]` instructions.
