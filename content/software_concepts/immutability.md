+++
date = 2019-07-21
title = "Making it Immutable"
description = "What is immutability and how it can help your development"
[taxonomies]
tags = ["immutability"]
blog_series = ["software concepts"]
+++

Mutability in terms of software usually will describe the ability to change the internal state of an
object once it has been created. Many main design decisions on frameworks and conventions favored
having mutable objects and didn't really touch on the concept of immutability. The clearest example
of this is the POJO concept in Java, or more specifically the notion of getters and setters.

## The Notion of Getters and Setters

I will take a short tangent for those not familiar but if you are please skip ahead.

In a typical Java object you will encapsulate fields by making them `private` and unable to be
directly accessed. The access to these internal fields are usually protected by getters and setters,
which simply get the current value and update the field respectively. A basic example for getting
and setting the name of a person can be seen below.

```java
public class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

## Why the Change of Mind?

So if we made all these decisions in languages and frameworks that are now so popular why should I
even need to know this? Well this may be a little bleak but in software just because something is
popular unfortunately does not mean it is good or right. I would almost argue that it is very hard
to find something good in software because designs can be so subjective and things simply move so
fast that we can suddenly do things that we couldn't do when our first decisions were made.

My favorite comparison is comparing older languages to more modern languages. A simple example is
Java vs Kotlin. Whilst these both live in a similar realm of the language space I don't believe you
can justly compare them as they both came from different times when we knew different things. Java
is much older than Kotlin, which was a language specifically designed to bake in all the Java best
practices into a language. The fact that we can make an entirely new language with the goal to
simply enforce best practices is a clear sign that as an industry we discover new and better
things arguably daily.

### Immutability is now part of the language

In this example one of the things that Kotlin has added that Java didn't have (at least when Kotlin
first came out) was the notion of immutability. In Java (and many other languages of the time) the
notion of immutability is not as evident. It has the `final` keyword to enforce variables are set
only once but it is an extra keyword always needed to be added and certainly not what you learn in
most beginner courses. Whereas more modern languages have introduced separate keywords for a mutable
and immutable variable. This makes the thought about this variable's mutability far more apparent.

Let us take a really basic example of getting a result back from a method invocation. In Java I can
simply make the types line up and we are good to go. I can almost forget about the mutability
question because it isn't required at all. In fact this happens a lot for me so I have actions
performed on save to add the `final` keywords for me.

```java
String result = myObject.invoke();
```

But if I want it immutable I need to remember my `final` keyword:

```java
final String result = myObject.invoke();
```

Whereas in Kotlin, thanks to advances in what we can achieve within the compiler the notion of
making the types line up is not as important and the developer's responsibility is to choose the
correct keyword to define the mutability of the result.

If the result is to be immutable

```kotlin
val result = myObject.invoke()
```

If the result is to be mutable

```kotlin
var result = myObject.invoke()
```

As you can see from this example, the notion of immutability has become more prevalent in the
developers decisions. To the point where the compiler can tell you that the mutable variable you
made could actually be mutable.

### Mutability can bite

So we have explored how the notion of mutability is now brought closer to front in terms of our
development, but still have not answered why? The answer is pretty simple, making everything mutable
and not addressing that upfront led to a large cognitive load for developers and with that bugs will
follow. The issue is embedded in the fact that it lead to objects having vast amounts of way of
changing state. The more possible states an object could be in the harder it is to reason with and
that is what happened.

An immutable object is one that is set at construction and has no way in which it is able to change
afterwards. Of course this does come with a few issues you need to keep track of and I will discuss
those later. The main point I want to express here is that mutability can cause difficult to manage
code and we should be aware of our decision to use mutable or immutable objects.

## Converting to Immutable Objects

So we have hinted at why you might want to try immutability but how can you do it? Every language
has its own way to manage immutability and the extent to which you can achieve immutability. We will
stick with Java for the time being. Let's return to the person example before but make it a little
more interesting. Let's say the person has a name and a phone number and we want to make it a basic
immutable object. It would look something like this:

```java
public final class Person {
    private final String name;
    private final String phoneNumber;

    public Person(String name, String phoneNumber) {
        this.name = name;
        this.phoneNumber = phoneNumber;
    }

    public String getName() {
        return name;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }
}
```

The key points to note:

- Whilst not exactly data immutable, having the `final class` means that the class is immutable and
  cannot be extended
- `private final` variables ensure they are set once and only available within the class scope
- There are no setters, only getters

### Not Everything is Perfectly Immutable

Whilst the above example is the closest we can get to perfect immutable in Java there are some
things to consider with these, since Java as a language chose mutability as the default.

One of the biggest confusions is that following those steps above is all you need for it to be
immutable but let us consider an extension on the example. Let's say that this person object now
needs to cater for multiple phone numbers and my naive implementation is to do this by using a list.
Perhaps it could look something like the following:

```java
public final class Person {
    private final String name;
    private final List<String> phoneNumbers;

    public Person(String name, List<String> phoneNumbers) {
        this.name = name;
        this.phoneNumbers = phoneNumbers;
    }

    public String getName() {
        return name;
    }

    public List<String> getPhoneNumbers() {
        return phoneNumbers;
    }
}
```

Now this looks pretty good, all our data is only set once and we only have getters. The problem here
is that a `List` is not an immutable object whereas it just so happened in our earlier example
`String` was. Let's delve a little deeper into how immutability breaks here.

The client code for the above `Person` class could look something like this:

```java
List<String> numbers = person.getPhoneNumbers();
numbers.add("555444333");
```

Despite the poor choice of example phone number there is also something else concerning here, the
client is modifying the list of phone numbers. Now sure enough because the `Person` class returns
the reference to their internal field this is going to modify the state of the original `person`
object.

Let's have a look at what I mean by that (the comments on the right show the value printed)

```java
Person person = new Person("Frank", new ArrayList<>());

System.out.println(person.getPhoneNumbers().size()); // 0

List<String> numbers = person.getPhoneNumbers();
numbers.add("555444333");

System.out.println(person.getPhoneNumbers().size()); // 1
```

So even without any setters we can see that we have still actually written an interface that allows
mutability, but we really want this to be an immutable object so how can we do this? Unfortunately
this does involve a little bit of work in Java but certainly does give us the immutability benefits
so if it is what you are working for then definitely the cost is worth it.

The general practice is you need to return the value not the reference. For a list this is by making
a copy of the list and returning that.

*Warning*: Making a shallow copy may not always be enough, if the contained object is not immutable
then a deep copy is necessary.

## Things to be aware of with Immutability

Nothing is a silver bullet and immutability certainly has some caveats to be aware of, here are a
few obvious ones:

1. If the type returned from the getters is not immutable itself, then your class has just become
   mutable. (Refer to above section)
1. Java does support reflection so people can get fancy with this.
1. A lot of frameworks have been built based on mutability so can be difficult to integrate
1. Can create many copies of objects which can be costly

The last one is particularly important and one of the main reasons immutability wasn't popular
earlier. Having pure immutable objects means that you need different copies for every possible value
which can consume a lot of your memory resources. Obviously we have a lot more memory we can use
nowadays but that does not mean we can disregard this limitation. When making objects immutable
consider this, particularly when needing to perform deep copies as these are certainly not free.

## Benefits of Immutable Objects

Finally, why am I writing about this, why tell people to aim for immutability where possible? Quite
simply it makes me think less. In our profession as counter intuitive as that may sound I find it is
a big influencer because there will always be other difficult things wanting your attention.

If you have immutable objects you do not need to worry about shared state and who is modifying what.
This particularly becomes useful in concurrency. You can share as many immutable objects between
thread as you want and you won't have to worry about landing in a corrupt state because you cannot
modify the state of the object but only create a new one.

For the simplicity it has added to my development I would definitely recommend choosing immutable as
your standard and then consciously making the decision to make your classes mutable. This has even
become a standard for programming languages themselves, Rust is an example of such a language.

Regardless of if you agree or not on the value of immutability, I urge you to at least consider the
mutability facet of a class when you are designing and not let it be an after thought because the
language you develop in doesn't accentuate it.