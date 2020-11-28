+++
date = 2020-11-28
title = "Wrapping primitives in domain objects"
description = "It is easy to enjoy primitive obsession when managing domain objects but we should always consider wrapping these into their own class to make a clearer and more developer friendly codebase"
[taxonomies]
tags = ["kotlin"]
+++

An interesting pattern I have seen with many developers is that they shy away from creating classes
to represent domain objects, especially basic ones. This is pretty common for identifiers that are
numbers, UUIDs, or just strings. Another similar pattern I see is for booleans, there is plenty of
code out there that uses a boolean value to represent some binary notion that does not really map to
a true or false. A tell tale is if there is a variable or function starting with `isXXX`. This is by
no means a rule but should trigger a line of questioning whether a boolean is the correct data type.
That is exactly what I wanted to explore in this post, rich data types and being able to add a whole
new level of comprehension to your code and how it fits into your application architecture. This is
so common it has been given a name of [primitive
obsession](https://refactoring.guru/smells/primitive-obsession).

## Wrapper classes

Using a customer ID as an example I wanted to show some ways in which we could represent this
instead of its primitive type and what values we get from it. The way I manage these domain objects
is what I call wrapper types or classes but there are several names I am sure, essentially I am making a class
that wraps the primitive type that represents the value.

To start off this example lets assume we represent the customer ID as an integer, which is pretty
common if we use the auto incrementing field of a SQL database. The use case we will use is a
service that stores the customer movie recommendations and has an API to read the recommendations
for a given customer ID, it may look something like this (depending on the framework you use):

```kotlin
@Controller
class RecommendationController(private val repo: RecommendationRepository) {
    @Get
    fun getCustomerRecommendations(@Path customerId: Int): Preferences {
        return repo.findWithId(customerId)
    }
}
```

The first and most common opposition I hear is what value does it add? Of course in the trivial
example I have provided above it is hard to see the value, but as your application grows it becomes
more valuable and it is much easier to be using it from the outset. Rather than explain all the
benefits of it over the costs of an extra class I would like to show it through a set of examples
similar to what I have seen/dealt with myself.

### Clearer function signatures

Let us now consider a little more complicated example, because we could be giving a customer
thousands of recommendations we allow the client to provide a limit to the number of recommendations
we return since they likely do not require all at once. This will again be another `Int` given to
the controller as a query parameter. Another common feature here is that when you have an account
with your movie streaming company you can actually set up some profiles so lets also add that since
we will likely want to be getting the recommendation for that profile only, so now our controller
function looks a little more like this:

```kotlin
@Get
fun getCustomerRecommendations(@Path customerId: Int, @Path profileId: Int, @Query limit: Int) {
    // ....
}
```

Hopefully you can already see that we now have several different domain elements that are all
represented as an `Int`. In the controller layer (the outer layer) this makes perfect sense as we
want to keep these simple types to work nicely with the framework for deserialization however our
internal API of the repository now will have 3 input parameters that are all of the same type. **If
you do not use wrapper types you have to rely on the parameter names of the API to know which goes
into which place**, if we were to represent these with their own classes instead the compiler will
tell us when we have put the profile ID in the place of the customer ID.

### How to represent the wrapper class

Now that we have our first good example why it would be worthwhile adding these classes lets see
what it will look like. This will greatly vary between languages and the way I am representing it
here is by no means the best or only way it will all depend on what actions you need to perform on
the type. In Kotlin there are few options and I will go through them all and why I choose some over
others.

#### Inline classes

The absolute best option in Kotlin is [inline
classes](https://kotlinlang.org/docs/reference/inline-classes.html) as this use case is exactly what
the feature was built for, however as of the writing of this post it is still in experimental stages
so I have not used it a lot on production code bases but definitely is fine to have a play with if
you love to be on the bleeding edge.

```kotlin
inline class CustomerId(val value: Int)
```

#### Data classes

Typically I lean on [data classes](https://kotlinlang.org/docs/reference/data-classes.html) as these
objects contain some piece of data as the name suggests. The benefit here is mainly that the
`equals` function is generated for us which is something I commonly find that I need to make use of.
The only element of this implementation to be cognisant of is that you will want a consistent naming
convention and expect direct access to the inner field as data classes require there to be a public
accessor. Personally I opt for `value` (and the generated Java of `getValue`).

```kotlin
data class CustomerId(val value: Int)
```

When the domain object is represented as a `String` I also adjust the `toString` function to just
provide the value rather than the generated form which may be unexpected, such as below. This of
course entirely depends on your use case and may not always add value for you.

```kotlin
data class CustomerName(val value: String) {
	override fun toString(): String = value
}
```

#### Normal classes

If there is not the use case for `equals` **and** you do not want to allow for direct access to the
inner field I tend to just write a typical wrapper object very similar to the data class but with a
slightly different API. Honestly I have not seen a lot of use for this case in a Kotlin only project
but in Java and Java/Kotlin projects I have found it valuable to represent absence of a field, for
example for a certain request the customer age may not be required:

```kotlin
class CustomerAge(private val innerValue: Int?) {
	val value: Optional<Int> = Optional.ofNullable(innerValue)
}
```

In an entire Kotlin project the above does not make a whole lot of sense as nullability is built
into the language however I wanted to show it for completeness as you may be working in a language
that does not or you may have a use case where you do not want to allow for direct access to the
value for some reason.

## Benefits of wrapper classes

To finish up I wanted to go over some of the benefits I have seen for wrapper classes.

### Clearer intent

Wrapper classes make it crystal clear what type of data you are working with, you will never have to
trace the call stack or hope that someone wrote the currently correct variable name for the
arguments for the string you are working with. The class now tells you exactly what it represents
and the intent on how it should be used. The most significant impact for me has been when reading
older code, it makes it much easier to read function signatures and the logic as I can lean on the
types to inform me whereas when things are all primitive types I rely on the developers variable
naming which has many times become outdated or simply confusing.

This is most apparent when dealing with booleans. Consider the case where content that a customer
can see is restricted based on their subscription plan. A pretty common thing to see is:

```kotlin
val isAllowed: Boolean = doesCustomerHaveSubscription(customer) && subscrptionCanAccess(customer, content)

if (isAllowed) {
	// Do something...
}
```

There are several other variations of this where we misrepresent a binary state as a boolean,
allowed/restricted, valid/invalid are a couple. By representing this as a boolean, certainly we do
get a lot of benefit because we can leverage the language construct for combining these evaluations
but it is very easy to start evaluating either side of the binary choice. For example recently I
have reviewed code where in one spot we are checking to see if it is allowed and at another location
not far away we are checking to see if it is restricted. I can speak from experience that this is
very difficult to understand and keep in a mental model.

Further, there is always some intrinsic element of the domain that dictates how these values can be
combined that do not always map to the same combination as boolean logic, or you have forced it to
match. For example, we typically have it that to be valid when combining evaluations, all must be
valid, if one is invalid then the whole combination is invalid. Certainly this does map to having
valid as true and invalid in boolean logic with the AND operator by why represent it like this when
we can represent it with much clearer intent:

```kotlin
sealed class Evaluation {
	abstract fun combine(other: Evaluation): Evaluation
}

class Allowed: Evaluation {
	override fun combine(other: Evaluation): Evaluation {
		return when(other) {
			is Allowed -> Allowed
			is Restricted -> Restricted
		}
	}
}

class Restricted: Evaluation {
	override fun combine(other: Evalution): Evaluation {
		return Restricted
	}
}
```

*Above I have used [sealed classes](https://kotlinlang.org/docs/reference/sealed-classes.html) which are extremely helpful for representing a known finite number of states of a value.*

Then we can achieve the exact same as above but getting the intent much clearer, certainly there is more code but that is a small price to pay for ease of reading:

```kotlin
val contentEvaluation: Evaluation = evaluateCustomerSubscription(customer)
							.combine(evaluateSubscriptionAccess(customer, content))

if (contentEvaluation == Allowed) {
	// Do something...
}
```

### Easier refactoring

As with a lot justifications, adding a wrapper class makes the code base easier to adapt to change,
change in design or requirements. There are several examples for this but I wanted to share one that
has stuck with me, let's imagine we have an application that stores a customer's address and one of
the fields of the address is the zip code and since we are an American company we know that zip
codes in USA are just numbers so we decide to model this as an integer. Then we may have some
elements as follows around the code base:

```kotlin
data class Address(val streetName: String, val zipCode: Int)

// ...

fun getShippingCostForZipCode(zipCode: Int): Cost

// ...

fun generatePostalSlip(name: String, zipCode: Int): String
```

Now consider the case where the company wants to expand to somewhere like the UK which has letters
in their zip code, suddenly we now need to represent the zip code as a `String`, so we need to go
into all these locations and change them up. If however we had chosen to represent this with its own
wrapper class then it would be pretty simple, we can just change the internal value of the class and
adjust the functionality as required. I know that we have great IDE tooling that can manage this but
that still does not eliminate the large diff that is typically created which will be reviewed or
examined at some later point in time. Having a smaller diff makes it much easier for your colleagues
to understand the actual purpose of the change. In the above example you will be conflating the
adding of the UK marketplace with changing the zip code to a string.

### Richer API

It is obvious that we have no good reason to add 2 to a customer ID, yet when we simply represent a
customer ID as an integer this is in fact part of the API we provide. Further how many times have
you seen private methods acting on a string to slice it a certain way or format it just so, and then
when you have used it enough times you get the idea to add this to some distant ambiguous "util"
class? This to me is a smell begging us to make some sort of class around this data to expose an API
that matches the domain context and this is precisely what a wrapper class will provide for our
data. One very common one is a name, we see it everywhere represented as a string and then have
utility functions capitalizing the names, getting the first and second name, etc.

```kotlin
fun printNameForLetter(name: String) {
	println("${firstNameInitial(name)}. ${capitalizeLastName(name)}")
}

private fun firstNameInitial(name: String): String {
	// ...
}

private fun capitalizeLastName(name: String): String{
	// ...
}
```

Such functions are far better suited to its own class where all of this common functionality can be
localized and benefited across all use cases.

```kotlin
class Name(value: String) {
	val fistNameInitial: String = ...
	// ...
}
```

The latter allows for far more reuse due to the proximity and easier to manage any changing
requirements. One great feature of Kotlin it would be naive of me to skip is that it provides
[extension functions](https://kotlinlang.org/docs/reference/extensions.html) which bridge this gap
and make for a very different design pattern. I am personally still working out the best conventions
as to when add a function to the class or as an extension function. Currently my rule of thumb is if
it will be used elsewhere add it to the class, simply because then when looking for the definition
it is all in one place rather than across several files as extension functions.