+++
date = 2017-09-15
title = "Writing Rust Documentation"
description = "A very simple reference to help those wishing to add to the Rust documentation or for the documentation of your own crates."
+++
Personally I have found Rust documentation to be great and very easy to follow.
Further I really enjoy that I can easily access the source code from the
documentation.

Recently I have tried to contribute back through writing some documentation on
the standard library and thought I would try construct a reference for
guidelines and principles the docs team seem to be following.

## Formatting

* Wrap lines to 80 characters
* Leave room for headings to breathe, empty line above and below.

## Templates

Standardization is always good in the standard library so to address these the
documentation team developed some templates for common documentation which makes
the feel of the documentation very consistent and easy to write.

### Iterator Template

When describing a struct that implements `Iterator` the following template
should be used:
> An iterator that {what it does}
>
> This struct is create by the {foo} method[ on {trait}]. See its documentation
> for more.

## Linking

This was the part I continue to struggle with as it is a bit confusing without a
reference nearby.

### Modules

Each module is located in its own sub-directory. That is if my crate is called
`foo` with a public module `bar`. Then to reference something in that module my
comment link would need to be something like:
```rust
/// [bar::MyStruct`]: bar/struct.MyStruct.html
```

### Lexical Elements

#### Structs

All structs will be prepended with `struct` such as `struct.MyStruct.html` and
will be located on their own HTML page.

#### Enums

All enums will be prepended with `enum` such as `enum.MyEnum.html` and will be
located on their own HTML page.

#### Traits

All traits will be prepended with `trait` such as `trait.MyTrait.html` and will
be located on their own HTML page.

#### Functions

Functions will always be related to some struct, trait or enum, hence will not
have their own HTML page and need to be referenced by ID. So let's say I have a
function called `my_func` for a struct called `MyStruct` (I know pretty original
over here), then to reference this would be:

```rust
/// [`my_func`]: MyStruct.html#method.my_func
```

However if you were wishing to reference a function on a trait it would be
slightly different.

```rust
/// [`my_func`]: MyTrait.html#tymethod.my_func
```
