+++
date = 2020-03-22
title = "Kotlin class validation in initialization"
description = "Kotlin introduces a different syntax for initializing a class over Java. When we had some validation in the constructor the migration may be a little difficult."
[taxonomies]
tags = ["kotlin", "validation"]
+++

Recently I was working with a bit of code where we had written validation logic in the constructor.
This was simply validating that the input received from an API call matched what we required. We are
currently migrating some of the code across from Java to Kotlin and whilst most of the time this is
a straightforward process this validation one wasn't so smooth.

Let's say you have a `FullName` class that requires both a first and last name that cannot be empty.
We might have a class something like the following:

```java
public class FullName {
  private final String firstName;
  private final String lastName;

  public FullName(final String firstName, final String lastName) {
    if (StringUtils.isBlank(firstName) || StringUtils.isBlank(lastName)) {
      throw new InvalidDataException("Names must not be blank");
    }
    this.firstName = firstName;
    this.lastName = lastName;
  }
  // getters...
}
```

We want to move this across to a new `FullName` Kotlin class instead, below are some of the options
we managed create. Each have different styles and benefits so it is up to you which you prefer.

*For these examples I am assuming that `firstName` and `lastName` didn't have to add a `null` check.
If this is not the case for you simply change the type to a nullable type and perform the check
using [safe calls] or similar. Do this so that you do not get an exception throw from the standard
library which is essentially a null pointer exception.*

### Using an `init` block

```kotlin
class FullName(firstName: String, lastName: String) {
    val firstName: String
    val lastName: String
    init {
        if (firstName.isBlank() || lastName.isBlank()) {
            throw RuntimeException("Name cannot be blank")
        }
        this.firstName = firstName
        this.lastName = lastName
    }
}
```

**Try this [in the playground][play1]**

#### Pros

- This has the same interface as the previous Java class so all calling code should fit nicely

#### Cons

- We haven't been able to convert to a `data class` where it may have been similar to one in Java.
  This is because a `data class` requires a public field being present in the primary constructor.
  This can easily be overcome by using your IDE to generate the `equals/hashcode` and `toString`
  functions you may have wanted.

### Use a static constructor

Using static constructors is a recommended practice as per Effective Java, so one alternative is to
put the validation logic into such a function so the class becomes simple. In our above I believe it
was done in the way above to match the Spring MVC convention.

```kotlin
data class FullName private constructor(val firstName: String, val lastName: String) {

    companion object {
        fun of(firstName: String, lastName: String): FullName {
            if(firstName.isBlank() || lastName.isBlank()) {
                throw RuntimeException("Name cannot be blank")
            }
            return FullName(firstName, lastName)
        }
    }
}
```

**Try this [in the playground][play2]**

#### Pros

- We are now able to use the `data class`
- The class itself doesn't have a 'smart' constructor
- We are able to make all construction through the static method using `private constructor`

#### Cons

- We have now changed the signature of the class for the callers of the existing class

### Using `Invoke` to get both

I have not been able to test how this works with Java interoperability but this option will give you
the best of the above two solutions.

```kotlin
data class FullName private constructor(val firstName: String, val lastName: String) {

    companion object {
        operator fun invoke(firstName: String, lastName: String): FullName {
            if(firstName.isBlank() || lastName.isBlank()) {
                throw RuntimeException("Name cannot be blank")
            }
            return FullName(firstName, lastName)
        }
    }
}
```

**Try this [in the playground][play3]**

[safe calls]: https://kotlinlang.org/docs/reference/null-safety.html#safe-calls
[play1]: https://pl.kotl.in/0dQNj7Fb2
[play2]: https://pl.kotl.in/QfFA6uqJt
[play3]: https://pl.kotl.in/GIfzAHuXR