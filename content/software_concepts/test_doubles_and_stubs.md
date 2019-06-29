+++
date = 2019-06-19
title = "Making your own test doubles and mocks"
description = "Exploring the difference between test doubles/mocks and stubs. In particular looking at how we can make them without a framework"
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

<!--

TODO: A picture would be pretty handy here

Example will be database interaction where we need to ask a certain query based on parameters and
process the output to either return a list of a sub set of results or the number.
-->

Let's try and put this into action with an example. Let's say you have a little application that
manages your books and one of your services looks up those books by author and returns the list of
book titles. However our book model is pretty complex so we want the service to map this to a simple
model of title and author when it has retrieved the results.

Our database entity may look something like this:

```java
public class BookEntity {
    // Note: this is bad practice to give these primitive types but will suffice for this example
    private String title;
    private String author;
    // ...some more fields

    public String title() {
        return title;
    }

    public String author() {
        return author;
    }

    // ... some more methods
}
```

With our output model like this:

```java
public class BookModel {
    private String title;
    private String author;

    public BookModel(String title, String author) {
        this.title = title;
        this.author = author;
    }

    public String title() {
        return title;
    }

    public String author() {
        return author;
    }
}
```

Then our service may look a little like:

```java
public class BookLookupService {
    private BookRepository repository;

    // ...

    public List<BookModel> findByAuthor(String author) {
        List<BookEntity> bookEntities = repository.findBooksByAuthor(author);
        return bookEntities.stream().map(x -> new BookModel(x.title(), x.author())).collect(Collectors::toList);
    }
}
```

Now let's get onto writing these tests!

## Testing with a double

As I said before we can split the interaction up into 2 whilst writing our unit tests for the
`BookLookupService`. We will use a test double to ensure that it communicates with its
`BookRepository` collaborator correctly. For this to work `BookRepository` should be an interface
that we can implement for our test, as all `BookLookupService` needs is a `findBookByAuthor` method.

This is perhaps what a possible double could look like.

```java
public class BookRepositoryDouble implements BookRepository {
    int callCount = 0;
    String authorArgument;
    // ... implements other methods of the interface

    @Override
    public List<BookModel> findBooksByAuthor(String author) {
        callCount++;
        authorArgument = author;
        return new List<>(books); // Return some books we set earlier
    }
}
```

As you can see we are interested in how our `BookRepositoryDouble` is being called. Hence we track
the parameters given and times called so that we could make some assertions on those values. On this
side of the collaboration we want to see that `findBooksByAuthor` is called with the correct
parameters. Arguably in this example the test doesn't provide us much but the concept extends to all
cases.

```java
@Before
public void setUp() {
    service = new BookLookupService(mockRepository);
}
@Test
public void checkCorrectCall() {
    service.findByAuthor("Author Name");
    assertEquals("Author Name", mockRepository.authorArgument);
}
```

## Testing with a stub

On the other side we want to ensure that our service interprets the received data correctly. For
this case we create a stub where we don't track what is given but return something of value we can
test. For example we would want to test the nominal path where there are a few books but we would
also want to check how our service works when there is only one book or no books. As you can see the
test scenarios with the stub are quite different than that of the double.

```java
public class BookRepositoryStub implements BookRepository {
    // ... implements other methods of the interface

    @Override
    public List<BookModel> findBooksByAuthor(String author) {
        return new List<>();
    }
}
```

Again we want to split our tests up as best we can to only have one assertion per test so there can
only be one cause of failure.

```java
@Test
public void checkWithTwoBooks() {
    service = new BookLookupService(twoBookRepository); // Instantiate with stub returning 2 books
    List<BookModel> result = service.findByAuthor("Author Name");
    assertEquals(2, result.size());
}

@Test
public void checkWithZeroBooks() {
    service = new BookLookupService(noBookRepository); // Instantiate with stub returning 0 books
    service.findByAuthor("Author Name");
    assertEquals(0, result.size());
}
```

## Frameworks

I just wanted to finish up this piece discussing the elephant in the room. Why are you not just
using a framework to do all this? Honestly as I was writing these code examples I was thinking much
the same. There is a lot of boiler plate and repetition, surely we could do better. However the
problem with jumping straight into using frameworks is you don't get the chance to understand why
they exist and instead only learn their usage at surface level. Hopefully after going through these
examples when you next grab a mocking or testing framework you can appreciate and understand what it
is doing for you rather than just blindly following examples because it sure does make a lot less
code to write!