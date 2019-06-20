+++
date = 2019-06-19
title = "Making your own test doubles and mocks"
description = "Exploring the difference between test doubles/mocks and stubs. In particular looking at how we can make them without a framework"
draft = true
[taxonomies]
tags = ["testing"]
blog_series = ["software concepts"]

+++

This is another topic I was wanting to cover whilst I explored some topics that usually are missed
in schooling. Test doubles or mocks and stubs. The first thing I want to note is that I don't intend
on using any frameworks as I am aiming to try explain the concepts so hopefully it becomes apparent
as to why the frameworks came in place. The next is how we will look into this, to start I want to
clarify my understanding for the difference between the stubs and doubles and what both of these
enable. Then I want to consolidate these with some examples, so let's get into it.

## What exactly is the difference?

Now when I was first introduced to mocking as it is called I wasn't told of the difference and the
value, instead it was "Here is a cool framework that does it all for us". Don't ask how it works or
what it does, it gets our tests to pass so we move on. What I came to learn later is that the
framework was blocking me from understanding what exactly was going on. What was worse, the
framework obfuscated bad design choices because it made it simple to test my classes even when they
shouldn't have been. That was when I was told of *test doubles* and *test stubs*.

These two concepts are rarely seen too far from each other. They form a very valuable part of being
able to write good unit tests that are able to refine your system under test, where for the examples
below that will be a single class but in different scenarios this could vary. It essentially boils
down to your system under test needing to interact with some other system, who we will call its
collaborators, and we want to test these interactions.

When your service interacts with another service it can be broken down into 2 key parts. The request
and the response. When unit testing this we want to ensure that what we request of our collaborators
is correct and when receiving a response we correctly manage that. Therefore this can lead to two
separate tests for each interaction so to keep a single assertion per test. When we test the
**request** we will use a **test double** to assert that we request the correct behavior from the
collaborator. Then on the return with the **response** we will use a **test stub** to provide some
dummy data that our service can process.

TODO: A picture would be pretty handy here

Example will be database interaction where we need to ask a certain query based on parameters and
process the output to either return a list of a sub set of results or the number.

## Testing with a double

## Testing with a stub