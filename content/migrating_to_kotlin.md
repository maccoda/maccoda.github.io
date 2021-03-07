+++
date = 2021-03-07
title = "Migrating your project from Java to Kotlin"
description = "Sharing some experiences around migrating a project focusing on tricks to make it easier and traps to avoid."
[taxonomies]
tags = ["kotlin"]
+++

I have been part of a few migrations of projects from Java to Kotlin in
particular as I have convinced more teams of its value over Java. However I did
not want to write another comparison article but rather assuming that you are
undertaking this migration how you can tackle it. One of the great benefits here
is that Kotlin is entirely interoperable with Java allowing you to not make the
changes in a single grand rewrite effort. Instead you can address the changes
piecemeal.

This post has been split into two sections of **traps** to avoid when migrating
and **tricks** to try. Not all sections may be relevant to your particular use
case as some sections are focused on the tooling/frameworks that has been used,
so please find the ones most applicable and hopefully it assists.

## Traps

First we shall cover the areas of migrations that teams have tripped up on
unexpectedly and how they were overcome.

### Compilation order

When working with a polyglot project it becomes important to understand the
compilation order as this defines the allowed dependencies. When using tools
such as Gradle I believe it is configurable using the `dependsOn` argument of
the task. In my experience the standardised tooling has chosen the order to be
Kotlin => Java and thus I shall cover the elements that this direction of
dependency introduces.

The output class files should be co-located and hence full interaction should be
capable however there are many fancy tools in the JVM world that generate code
or hook into the compilation cycle to alter the byte code (such as Lombok in the
next section). It is important to know how these tools works in principle but
also as it can lead to unexpected issues. Therefore comprehensively understand
the tooling of the project prior to the migration to determine any of these such
tools used in the Java toolchain as these will not be able to be used by the
Kotlin classes as they are compiled prior to the tool being invoked.

### Lombok

[Lombok] is such a tool that hooks into the
compilation cycle to generate boilerplate code within Java. Commonly used to
generate getters, setters, and constructors. These files are generated as part
of the Java compilation step, therefore all the Lombok generated code becomes
inaccessible by Kotlin code in a polyglot project. This has led to several
compilation errors of the nature of "symbol not found". Now that you are aware
of this trap in the migration these cryptic messages should ideally have a
clearer cause.

The best way to avoid this issue is to "delombok" your code as you migrate. It
is not necessary to "delombok" the entire code base or even the entire file. The
issue can simply be rectified by writing the required function explicitly as
Lombok skips over the generation of functions that already exist.

The following shows an object of performing just that, assume that our Kotlin
code only needs access to the `getName()` function.

```java
@Value
public class Person {
  private final int id;
  private final String name;
  private final int age;
}
```

Will be changed to the following to allow Kotlin access:

```java
@Value
public class Person {
  private final int id;
  private final String name;
  private final int age;

  public String getName() {
    return name;
  }
}
```

#### Builder pattern

Another commonly generated pattern via Lombok is the builder pattern. This is
very useful in Java in particular to aide in readability when there are several
parameters to construct an object. In Kotlin such a pattern is not required as
the language itself supports this with [named arguments]. What this does mean
however is when migrating such a class from Java to Kotlin you must implement
the builder pattern within Kotlin as the Java classes cannot make use of the
named arguments and further it also reduces the amount that is changed in each
migration step.

The following is an example of how to implement the builder pattern for a basic
data class.

```kotlin
data class Name(val first: String, val last: String) {

  companion object {
    fun builder() = Builder()

      data class Builder(
        var first: String? = null,
        var second: String? = null
        ) {

        fun first(first: String) = apply { this.first = first }
        fun second(second: String) = apply { this.second = second }
        // You can do better handling of null than this but I have found this
        // sufficient until the I get to migrate the calling class to Kotlin
        fun build() = Name(first!!, second!!)
      }
    }
}
```

## Tricks

Next up we will cover some tricks discovered to make the migration easier, in
particular focusing on tooling that Kotlin already provides to make this
simpler.

### Order to migrate

Migrating a project can be a monumental task depending on the size of the code
base, therefore it is important just as any other large changes that it be done
in small, manageable chunks. The following is some general advice on how to
tackle this migration.

**Convert the value/data objects first**. The principle here is that they have
the fewest dependencies (ideally none), therefore moving these across should be
simple. Also, typically to migrate a class it is greatly beneficial that the
dependent (or imported) classes are already in Kotlin so none of the above traps
occur.

**Work your way outwards**. Once all value/data classes have been converted
choose classes that are closely related to it, ideally having only a few
dependencies, moving gradually out to the entry point of the program.

**Either convert or add functionality, never both.** This is strongly aligned
with the recommendation when refactoring (which converting a class is), that
adding new functionality whilst also performing a refactoring creates two
possible sources of error. In regards to migrating to Kotlin in particular this
leads to a bloat in code reviews making it very difficult to see the
functionality changes. Thus always perform the migration, then add the new
functionality.

### Kotlin compiler plugins

Kotlin chose a few differing paradigms from Java which results in certain
tooling being unable to work due to their leveraging of particular Java
paradigms. The most obvious example is mocking frameworks in Java make use of
the fact that they can create a subclass of the class to mock This cannot be
done in Kotlin as easily as for Java as Kotlin has closed classes by default. To
assist in this the Kotlin team have provided compiler plugins such as [all-open]
and [no-arg] to address these limitations when migrating. It also allows you to
continue to make use of the more mature tooling in Java if you wish to.

In saying this however I do recommend exploring Kotlin native
libraries/frameworks where possible as these do provide a much better
experience.

### Kotlin JVM annotations

Another major difference in the languages is that Kotlin does not have a notion
of `static`, its `companion object` is similar but not the same. Therefore when
interoperating with Java one may still wish to use static functions or static
constants. Typically these will be placed in the companion object and need to be
accessed through `MyClass.Companion.MY_CONSTANT`. There are annotations provided
with the Kotlin standard library to make this look more like a Java class such
as `MyClass.MY_CONSTANT` which then means that you do not need to change all
call sites of constants and functions.

For constants/static fields the [`@JvmField`] annotation can be used and for
static methods the [`@JvmStatic`] annotation can be used.

These annotations are not only there to provide aesthetics but also can improve
your code. For example using [`@JvmOverloads`] one can get all variations of a
function generated for a Java class to consume. This is incredibly powerful for
creating test data for example.

[Lombok]:https://projectlombok.org/
[named arguments]: https://kotlinlang.org/docs/functions.html#named-arguments
[all-open]: https://kotlinlang.org/docs/all-open-plugin.html
[no-arg]: https://kotlinlang.org/docs/no-arg-plugin.html
[`@JvmField`]: https://kotlinlang.org/docs/java-to-kotlin-interop.html#static-fields
[`@JvmStatic`]: https://kotlinlang.org/docs/java-to-kotlin-interop.html#static-methods
[`@JvmOverloads`]: https://kotlinlang.org/docs/java-to-kotlin-interop.html#overloads-generation
