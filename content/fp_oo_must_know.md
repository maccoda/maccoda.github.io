+++
date = 2021-06-05
title = "Valuable functional programming basics every developer should know"
description = "With many new programming languages seeing the benefits of being multi-paradigm it is becoming increasingly valuable to understand the basics of them all"
[taxonomies]
tags = ["functional programming"]
+++

For a long time there was an idealistic divide between functional and object
oriented programming and this was manifested in the languages that came through.
In the current landscape however you are spoilt for choice with languages and
this divide is closing with a lot of the most popular languages available taking
the best aspects of both paradigms. Kotlin is a great example of this where it
is initially based on the JVM and follows OO principles with classes, etc but
also introduces a lot of functional principles such as first class functions and
the scary monad (I am no expert in this but there are a lot of easy to
understand properties that you can make use of). Another example is Rust where
it cannot easily be classified as either paradigm but uses functional elements
such as using the `Result` type to promote functions always return a result, as
well as object oriented notions such as the dot notation of functions on a type.

## Monad Concepts

Monad is a term used a lot in functional programming and especially in the vast
theory behind it. I did not wish to delve into such theory here but rather the
useful functions that become available because we can treat some objects as a
monad. Of particular interest is working with collections or streams of
elements. The main reason I personally find these incredibly powerful is that
they provide a common language for which we can describe the intention of common
functions that can be combined together to achieve our end result. A lot of
these functions you can use in place of a `for` loop so next time you are
looking at iterating over a collection with a `for` loop consider if one of the
following constructs are what you need.

### `map`

The `map` function is a transformation of each element from type X to type Y.
Depending on the language you are using this may modify the elements in place or
create a new collection of the mapped values (in true functional style).

An easy example is say I have a collection of blog posts and I want to obtain
the titles of all these, we can achieve this by:

```kotlin
val blogPosts: Collection<BlogPost> = listOf(BlogPost(title = "post 1"),
                                             BlogPost(title = "post 2"))
val blogPostTitles: Collection<String> = blogPosts.map { it.title } // ["post 1", "post 2"]
// Without the syntactic sugar
// val blogPostTitles: Collection<String> = blogPosts.map { post -> post.title }
```

Let's break this example down a bit to what is happening here. The `map`
function takes a function as a parameter which we will call our **transform
function**. This function maps the input type to some other type (this can the
same as the input type). The `map` function then iterates through the input list
and creates a new list applying this function to each element.

### `flatMap`

The `flatMap` function is actually a combination of the above `map` function and
the `flatten` function (not covered in this post). The `flatten` function takes
a container of containers type and **flattens** it to just a container type.
This is quite abstract so lets take a simple example. Say you have a
`List<List<String>>` and you want a `List<String>`, this in the broad sense what
the `flatten` function does. In this example the container is the `List` type.

Knowing this it is quite simple to describe the `flatMap` function although it
may still be difficult to wrap your head around in actual applications. The
`flatMap` function will apply the transformation specified to the element in the
container and then flatten it. This is typically required when the
transformation function returns the same type as the initial type.

As an example say we have a type that represents the books a customer has read
and we want to print all of the books a group of customers have read (we are not
concerned about duplicates). Such data may be present as:

```kotlin
data class BookCustomer(val booksRead: List<String>)

val bookCustomers: List<BookCustomer> = ...
```

The end type we are wanting is a `List<String>` containing all the book titles.
Let us first try this without `flatMap`  to aide in seeing when to consider
using it.


```kotlin
val bookCustomers: List<BookCustomer> = ...

val booksRead: MutableList<String> = mutableListOf()

for (customer: BookCustomer in bookCustomers) {
  val booksTitles: List<String> = customer.booksRead
  booksRead.addAll(booksTitles)
}
```

The key thing that using these functional paradigms provides is built in
immutability. In the above example we are needing to create a mutable list to
get the result. We will see in the below version this is not required.

```kotlin
val bookCustomers: List<BookCustomer> = ...

val bookRead: List<String> = bookCustomers.flatMap { it.booksRead }
```

The transformation we are performing is mapping the `BookCustomer` to the list
of book titles they have read, then the flatten part of `flatMap` handles to
reduction of the list of lists into a single list. This leads us into the next
section on `reduce` which is the general pattern of `flatten`.

### `reduce` or `fold`

The `reduce` and `fold` functions are more general functions to reduce a container,
typically a collection, to a singular value. A simple example we can explore is
reducing a collection of integers to their sum.

Firstly, the difference between the two functions is primarily in the arguments
that the functions take. The function signatures are below to make it simple to
reference.

```kotlin
fun <T, R> Iterable<T>.fold(
    initial: R,
    operation: (acc: R, T) -> R
): R

fun <T, R> Iterable<T>.reduce(
    operation: (acc: R, T) -> R
): R
```

The primary difference between the two functions is that one takes the `initial`
value of the resulting type and one does not. It instead uses the first value as
the initial value. This subtle difference results in the `reduce` function
requiring a non-empty collection to work on whereas `fold` can work on an empty
list and just return the `initial` value.

A simple example to show how these functions work is by implementing a summation
function that operates on a collection of integers.

```kotlin
fun sumReduce(input: Iterable<Int>): Int {
  input.reduce { acc, cur -> acc + cur }
}


fun sumFold(input: Iterable<Int>): Int {
  input.fold(0) { acc, cur -> acc + cur }
}
```

<!-- Explain the arguments provided here -->
The interesting element of these is the closure passed of the form `(acc: R,
cur: T) -> R`. This function is essentially and accumulation function that
combines the elements of the collection to the final result. It takes two
arguments:
- `acc` which is the current accumulated result up to the current element
- `cur` which is the current element

Putting this in action:

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

val sum: Int = sumFold(numbers)
```

The accumulation function is applied to each element and then the accumulated
value is updated and passed along. Stepping through the above example would look
like this:

*Note this is for fold, if using reduce the first row would be omitted as there
is no initial value*

| `acc` | `cur` | Accumulation function result |
| --- | --- | --- |
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 3 | 3 | 6 |
| 6 | 4 | 10 |
| 10 | 5 | 15 |

<!-- Give a good real life example of reducing validations perhaps -->

#### More use cases

The power of these functions is how flexible they are, any accumulator function
can be provided allowing you to work with any type. If you were inclined you
could spend more time studying functional programming and see how this is the
basis of a lot of incredibly power combinator functions. For example we can
create `flatMap` using `reduce` and `map`. Since this is typically handled in
the standard library we will look at a more likely scenario to find in a

project.

The example we will go through is that of some validation framework which
leverages the following:

```kotlin
interface Validation {
  fun validate(input: Input): ValidationResult
}

enum class ValidationResult {
  VALID, INVALID
}
```

Imagine we have a collection of validators and we wish to implement some
functionality that requires that all validators to run on the `Input` object and
it returns `VALID` if **all** validators return `VALID` otherwise if even one
returns `INVALID` it returns `INVALID`. The `reduce` function can do exactly
this as we can evaluate all the validators and then *reduce* them to a single
result.

```kotlin
val validators: List<Validation> = ...
val input: Input = ...

val result: ValidationResult = validators
                                  .reduce { acc, cur -> if(acc == ValidationResult.VALID) it.validate(input) else ValidationResult.INVALID }
```


### `filter`

The `filter` function is another helpful function to work with collections which
filters the collection based on a provided predicate. If the term predicate is
new, it is simply a function that takes an input object and returns a boolean.

Using another book example let's assume we have the below data type for a book
and we want to obtain the list of all books we read, this can be done as below:

```kotlin
data class Book(val title: String, val read: Boolean)

val books: List<Book> = ...

val booksRead: List<Book> = books.filter { it.read }
```

## Not quite must knows

The above functions and types are extremely valuable in most modern languages.
The below types are useful to know but depending on your language choice you may
not actually use it but understanding the concept is valuable regardless as it
can still have a positive impact on the way you write your code.

### Option/Maybe Type

It is common to have to represent the possibility of some data not being
present. Commonly in languages this is represented by some null value but in
pure functional programming this is actually represented by a type possibly
called `Option` (such as in Rust) or `Maybe` (such as in Haskell). The core of
this type is to be able to reflect the absence of a value in the type system.
This can also be achieved with languages still supporting null such as Kotlin as
it surfaces the nullability to the type level.

Having a container type to represent the absence of a value allows the client of
the data to act on it safely without needing to check whether the data is
present or not, which is something unable to be achieved with a basic null value
alone.

```rust
let data = Some(2)
let no_data = None
let result_some = data.map(|x| x + 3) // Some(5)
let result_none = no_data.map(|x| + 3) // None
```

### Result/Either Type

In an OO landscape it is typical to handle errors through exception handling.
Pure functional languages require this to be represented in the type system,
which is quite a powerful model. This is again named differently depending on
the language but it is a container type that has two possible representations.
If it is called `Either` then this will have a `Left` and `Right` type, this is
more generic than the `Result` type as you are able to represent any arbitrary
type that is a union of two possible types. A more focused point is the `Result`
type used in Rust to represent explicitly an error as it does not have
exceptions. The two types of this union are `Ok` and `Err` (for error).

Again this notion of a type may not be largely applicable in the OO domain but
it does promote you to think of handling errors and state in a different manner
and representing this in the type system. This concept can easily be implemented
in Kotlin using sealed classes and does not need to just be used for error
handling but can be used to create any container class that a client can act on.
Again the goal here is having the container class allows the client to work with
the data irrespective of the underlying state.
