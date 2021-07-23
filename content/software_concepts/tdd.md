+++
date = 2019-07-21
title = "Intro to the basics of TDD"
description = "Introduction to some of the basic concepts of test driven development"
[taxonomies]
tags = ["TDD", "testing"]
blog_series = ["software concepts"]
+++

TDD is definitely quite the polarizing topic in my experience, whilst not
everyone may agree with it I believe it is always valuable to understand it and
develop your own opinion on it.

TDD stands for Test Driven Development and was made made popular by Kent Beck.
It is an option of development process that we as software engineers can
consider. A wispy way to describe it that instead of the typical practice of
writing tests after production code your write it up front. I don't know about
you but when I was first introduced to this I thought it was pretty crazy. Let's
explore it a little further are there is more to it than simply writing your
tests first.

## The TDD Cycle

A core part of TDD is the TDD cycle. This is a fairly simple cycle and it
dictates the exact development cycle one should take when implementing TDD.

**Write test - Write code - Refactor**

The scope of this cycle is to whatever you define your unit of test. In the
object orientated world this will typically match to your class and your unit
tests for said class. Let's delve into each of these steps before heading into
an example.

This cycle is also known as the *Red-Green-Green* cycle representing the state
of the tests at each stage.

### Write Test

First and foremost, you write a test for the behavior that you expect of the
class. Most importantly you expect this to be a **failing** test. Here of course
remember that a compilation error is also a failing test. This test just needs
to describe one piece of functionality and fail so that we can implement it and
make it pass in the next stage of the cycle.

### Write Code

Now that we have a failing test that describes a piece of functionality we
implement that functionality. The interesting part here, and where most people
struggle to adopt TDD is that your implementation should be as minimal as
possible. That is, your implementation should only make your tests pass and
nothing more. The purpose of this is to drive implementation details to be as
simple as possible, attempting to remove as much unintentional complexity as
possible. It is very common at the end of this step to be looking at your code
and not being very impressed, but stick with it because we will get there.

This stage is complete once we have an implementation that makes all existing,
and new tests being written green (or passing).

### Refactor

This is the time where we look at our code and we can make it something that we
are proud of. Apply all the refactoring tools from your toolbox but make sure
after every refactor you run all your tests and they all stay green. There is
not much more to it, there may various different changes to make but it will all
depend on the situation.

Once we have completed this step and all our tests are green now would be a good
time to commit (small and often). Then we start the cycle again by adding in a
new test and continue.

## Give It a Try

There is whole lot more that we can delve into for TDD practices and reasons as
to why it is good but this post is simply to give an introduction to the topic.
We will try give a better understanding with a basic example of a much loved
Fizz Buzz program.

To give a quick recap of the problem so you don't have to search it up, the Fizz
Buzz problem is given a number and returns *fizz* if it is divisible by 3 and
*buzz* if it is divisible by 5. If it is both we return *fizz buzz*, otherwise
return the number.

*The example I will show is written in Kotlin for something a little different
and we will only look on the function level, just for those looking for the
constructor that never was. Also I won't be using any fancy test framework just
good old fashioned JUnit.*

Let's write our first test then, when we give our function 1 we expect it to
return 1.

```kotlin
@Test
fun `when 1 should return 1`() {
    assertEquals("1", fizzBuzz(1)) // Compilation error
}
```

First and foremost this test will have a compilation error which is indeed a
failing test because we do not have the function yet written. Whilst this is not
a good example, the compilation error step is important because it allows us to
consider the API that we want to define. This is because the Fizz Buzz problem
has a clear input and output but you can imagine when you create a new class you
can use this as a chance to consider API options.

Now we move into the write code phase. First thing we do is get it compiling and
passing.

```kotlin
fun fizzBuzz(num: Int): String {
    return "1"
}
```

This is exactly the time when you need to fight the urge to try and over
complicate this as this perfectly matches the requirements of the cycle. The
tests we have written pass and that is that. Since we don't have much code here,
there isn't much to refactor so let's commit and move to our next step.

```kotlin
@Test
fun `when 2 should return 2`() {
    assertEquals("2", fizzBuzz(2))
}
```

Moving back to the implementation stage we see it starts get a bit weird but we
follow the TDD rules of only implementing enough to satisfy the tests to pass.

```kotlin
fun fizzBuzz(num: Int): String {
    return if (num == 1) "1"
    else "2"
}
```

Now we can refactor this a little here, keeping the tests green and then commit
again before we start writing our next test.

```kotlin
fun fizzBuzz(num: Int): String {
    return num.toString()
}
```

Now we encounter a different case, so let's write a test for it.

```kotlin
@Test
fun `when 3 should return fizz`() {
    assertEquals("fizz", fizzBuzz(3))
}
```

Back to the implementation step we go!

```kotlin
fun fizzBuzz(num: Int): String {
    return if (num == 3) "fizz"
    else num.toString()
}
```

Again you can clearly say, hey this won't consider all other cases! Yet again we
are only considering making the current tests pass. However you may start to see
that this could possibly take a lot longer than the traditional method of
implement followed by test, which is definitely a common argument against TDD
and in this case it definitely shows.

In my opinion that is because the speed of TDD here can be improved by defining
tests based on expected behavior, rather than for this case just incrementing
the input (particularly since not all of our problems will take a simple
integer). So let's do exactly that, let's consider the case where it is
divisible by 5.

```kotlin
@Test
fun `when 5 should return buzz`() {
    assertEquals("buzz", fizzBuzz(5))
}
```

Hopefully you can see this test will obviously be failing so let's make it pass.

```kotlin
fun fizzBuzz(num: Int): String {
    return if (num == 3) "fizz"
    else if (num == 5) "buzz"
    else num.toString()
}
```

We are almost there we have a few more behaviors we want to check, these are:

- A number divisible by 3 but not 3
- A number divisible by 5 but not 5
- A number divisible by both 3 and 5

So let's quickly iterate through these as hopefully by now you have the basic
idea. For brevity I will collapse all the tests into a single code block but
these would have been added and then implemented one at a time. The
implementation blocks I will separate though to make it clear.

```kotlin
@Test
fun `when num is divisible by 3 should return fizz`() {
    assertEquals("fizz", fizzBuzz(9))
}

@Test
fun `when num is divisible by 5 should return buzz`() {
    assertEquals("buzz", fizzBuzz(10))
}

@Test
fun `when num is divisible by 3 and 5 should return fizzbuzz`() {
    assertEquals("fizzbuzz", fizzBuzz(15))
}
```

Now to the implementation. After testing for divisible by 3.

```kotlin
fun fizzBuzz(num: Int): String {
    return if (num % 3 == 0) "fizz"
    else if (num == 5) "buzz"
    else num.toString()
}
```

Divisible by 5.

```kotlin
fun fizzBuzz(num: Int): String {
    return if (num % 3 == 0) "fizz"
    else if (num % 5 == 0) "buzz"
    else num.toString()
}
```

Divisible by both 3 and 5.

```kotlin
fun fizzBuzz(num: Int): String {
    return if (num % 3  == 0 && num % 5 == 0) "fizzbuzz"
    else if (num % 3 == 0) "fizz"
    else if (num % 5 == 0) "buzz"
    else num.toString()
}
```

Now of course you can then refactor this however best suits your needs (I would
prefer braces here, maybe consider using a `when` statement, etc). Then there
you have it, we have just implemented fizz buzz by TDD.

## Isn't this a bit much?

As I mentioned before one of the biggest deterrents I have heard around is that
this procedure can be so strict and slow down development cycle. Honestly this
is definitely true at first but of course you are learning something entirely
new, you don't expect to be incredibly effective in a brand new programming
language when you first start, so why expect much different when learning a new
process?

TDD is a new mindset as well as process so it definitely does take time. To top
it off it does some with some very strict rules making you think just joined the
military! This is exactly why for me personally I haven't been able to integrate
the explicit procedure in my working process, however I still believe there is a
lot of value to TDD.

For me personally I really like the aspect of having to think of the behavior
upfront rather than implementation, which is always a great thing to be focusing
on in testing. However sometimes I need to play around with API ideas and test
out some possible implementations which may or may not work. This is where I
have found TDD a bit harder to utilize.

Overall there are some great takeaways from TDD and it is up to you how much or
little you buy into it and use it.

## Wrapping it up

Something I hope you take away from this introduction of TDD is that it is a
process with more to it than just writing some tests upfront. There is a nice
little cycle (Test-Implement-Refactor) to guide you through the whole process
which has some great goals. It allows you to think about the behavior of your
system rather than its implementation, making it easier to test and maintain. By
only implementing enough to get it to pass it provides a framework to reduce
incidental complexity. As well as always providing you a chance to look and
consider any refactoring once each cycle, which is much better than having to
perform it at a larger scale.
