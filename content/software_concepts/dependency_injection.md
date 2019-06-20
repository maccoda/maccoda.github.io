+++
date = 2019-06-15
title = "Why would I even use dependency injection?"
description = "A short walkthrough of dependency injection and why you should use it"
[taxonomies]
tags = ["dependency injection"]
blog_series = ["software concepts"]

+++

I was inspired to write this post and hopefully a little series on some of these key concepts of
software as I was asked "Why would I need to use dependency injection?". This brought me back to
when I was first learning these industry staples that you never get taught learning computer
science. Due to this I was hoping to write a little series around my learning experiences and
understanding of these concepts because I have always found the more perspectives you get the easier
it is to form your own understanding. This is the first one for the series so let's see how far we
can go.

If anyone does read this, firstly thank you and I hope it helps! Secondly if there is something you
would be interested in hearing thinking raise an
[issue](https://github.com/maccoda/maccoda.github.io/issues) for lack of a better means for the time
being and I will hopefully get around to it sooner rather than later!

## What is Dependency Injection?

Alright now that the little preamble is over with lets start out like any good problem solver and
define the *What* of the problem. According to our good friends at Wikipedia we define dependency
injection as:

> In software engineering, dependency injection is a technique whereby one object supplies the
> dependencies of another object.

[Wikipedia](https://en.wikipedia.org/wiki/Dependency_injection)

So this is pretty good and correct, assuming you understand the concept to begin with. So let's try
break it down a bit.

I first learnt this with Java, so one simple trick to think of dependency injection is moving all of
the `new` keywords out of your classes. Now this is a pretty dumb thing to say because something
obviously has to instantiate your classes and that is entirely correct but the point of dependency
injection is to create that separation between where your classes and their dependencies are
instantiated and where you write the fun problem solving logic. So the concept of dependency
injection is being able to **give** your classes their dependencies rather than you class
instantiating its own dependencies. A term you will hear a lot around this topic is Inversion of
Control (IOC). You are now giving control of what your classes dependencies are to a class higher up
the chain.

Don't worry there is a lot of writing here and my description is arguably confusing in itself but I
will clarify this with some examples soon! So stick with me!

## How do I use Dependency Injection?

There are a lot of fancy frameworks that you can use for dependency injection, no matter what
language you develop in. However I have no intention of showing how to use those frameworks because
if you want to learn you should practice it on the bare metal. I always enjoy getting my hands dirty
with this (as much as they can typing on a keyboard) because I find that to be the best way to
understand, making the mistakes and finding the answers.

Saying that let's get onto our example. Since I am not feeling incredibly creative today I am going
to take the classic shopping cart example. Our task is to make a shopping cart class that will allow
our customers to buy some items. The code below is one possible way of doing this (although please
don't use this to implement an actual shopping cart).


```java
public class ShoppingCart {
    private CreditCard creditCard;
    private LineItems items;

    public ShoppingCart(long creditCardNumber, List<Product> products) {
        creditCard = new CreditCard(creditCardNumber);
        items = new LineItems(products);
    }

    public void placeOrder() {
        creditCard.charge(items.total());
    }
}

```

So as you can see this shopping cart implementation needs to have a `CreditCard` to charge and some
`LineItems` to define how much to charge. Therefore it is pretty easy to see that the dependencies of
`ShoppingCart` are `CreditCard` and `LineItems`. A pretty easy way to see this in Java is they will
be fields of the class, you really only define a field if you need to use it perform some tasks.

However there are `new` keywords here so you can see the dependencies are not injected into
`ShoppingCart`, rather `ShoppingCart` knows exactly how to create its dependencies. It knows it
needs a **credit card number** and a **list of products**. In fact this is exactly why dependency
injection is important because it means your class does not need to know how to create its
dependencies but only how to **use** its dependencies. Instead of providing the parameters to
construct the dependencies we should have instead provided the dependencies themselves and
constructed those elsewhere, perhaps like this:

```java
public class ShoppingCart {
    private CreditCard creditCard;
    private LineItems items;

    public ShoppingCart(CreditCard creditCard, LineItems items) {
        this.creditCard = creditCard;
        this.items = items;
    }

    public void placeOrder() {
        creditCard.charge(items.total());
    }
}
```

So we can now create it elsewhere

```java
public class Factory {
    public ShoppingCart shoppingCart(long creditCardNumber, List<Product> products) {
        return new ShoppingCart(new CreditCard(creditCardNumber), new LineItems(products));
    }
}
```

Now we have separated our creation code from our domain logic code. In doing this it has provided us
dependency injection as you can see our `ShoppingCart` no longer has any `new` keywords!

## Why Dependency Injection?

The most obvious question now is, "Why would I do that? That looks like more code!". This is indeed
correct we do have more code but the number of lines you have written is never a sole indicator of
the quality of the code. Instead what you should be looking at is "Has this made it easier for me to
change the code as requirements change?" or "Is this code easily testable?". Making our code use
dependency injection has allowed us to say yes on both of those fronts.

### Changing with Requirements or Design

The biggest achievement we have made is we are now able to develop to an interface. That is,
`ShoppingCart` doesn't need to know anything about how `CreditCard` or `LineItems` work under the
hood, or if they are even concrete classes. All it needs to know is that it can call `charge` and
`total` on them respectively. Therefore if we only supported one type of credit card and we needed
to add another one, so long as it implements `charge` for the `CreditCard` interface our
`ShoppingCart` need not change.

In a different direction, if the design of `LineItems` were to change and it needed something
different to construct itself, our `ShoppingCart` is now unaffected. The only place it needs to
change is where we create it.

### Testability

Something else that dependency injection has aided with is making tests easier to write. If we had
of kept it the old way, testing `ShoppingCart` would be near impossible without needing to charge an
actual credit card. To avoid this we would need to use reflection to inject a mock and capture all
the interactions, doable but more complicated than it need be. Now we can simply make a test double
that looks like a `CreditCard` and then capture the interactions on that.

## Finishing Up

Hopefully through this very basic example you can see how you can use dependency injection in your
current work as well as how it is helpful. As with a lot of these practices it is hard to see the
benefit in the small scale but once your system grows and the number of moving parts increases its
value becomes apparent.
