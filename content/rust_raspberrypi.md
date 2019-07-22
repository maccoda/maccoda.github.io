+++
date = 2017-06-03
title = "Binding Raspberry Pi Libraries"
description = "Tackling two main areas that Rust has greatly simplified. Binding to C/C++ libraries and cross compiling for embedded systems (or not too embedded in terms of the Raspberry Pi)"
[taxonomies]
tags = ["rust", "raspberry pi"]
+++
My background with software stemmed from my interest in electronics and embedded
software and when I saw that Rust was starting to get some traction in this area
I knew that I had to give it a go and see how far I could get.

To see the actual source referred to in this post you can find it on [Github][repo]

TL;DR Got the LED blinking. Boo yeah!

## The project

The goal of the project was create Rust bindings for the [wiringPi library],
this is a Arduino like library for using the GPIO on the Raspberry Pi. Starting
off this was just an experiment to looking into the usage of the `bindgen` crate
now that they have a very useful [guide] for how to utilize `bindgen` in
generating FFI bindings to C and C++ libraries. Of course as I should have
realized, this also would then lead to cross compilation again but now with the
target being an embedded device and hopefully opening the way for more of these
ports and making Rust a common language for ARM devices.

Also it would be extremely rude of me not to mention that this work has already
been done my Ogeon in the crate [rust-wiringpi], so this was more of an exercise
for myself, but I was very lucky someone had done the hard yards as this
repository greatly assisted me in the cross compilation aspect of this project.

## Bindings

Writing the bindings was rather painless thanks to the guide and `bindgen`, I
won't explain the process here as the [guide] does a great job of that. In
generating these bindings there was very few manual components in getting the
bindings created. However the bindings created aren't very Rust-ic, they are
just a one to one mapping of the functions and constants defined within a
header. It was really lacking a lot of the strong typing that I have come to
love in Rust, so I thought surely there is a better way to make these bindings,
and more importantly surely I am not the first to have this idea.

Sure enough I wasn't after doing some searching I found some interesting takes
on how to make the bindings to existing C/C++ libraries. The one I chose was to
add the library as a [git submodule] within my repository so that I was able to
build it from source, this would prove to be vital for the cross compilation
aspect of this. Then on top of this I had the one simple project that would
generate the bindings using `bindgen` then finally the main project which would
be using these bindings but expose an API of my choice rather than that of the
wiringPi library (not saying its API is poor, just wanted to make it more
idiomatic for Rust).

This meant that my structure would look something like the following:
```
wiringpi-rs
├-src
├-wiringPi-bindings
| ├-src
| ├-build.rs
| └ Cargo.toml
└-WiringPi
```

The most important aspect of wrapping these bindings also means that users of
the library aren't having to wrap every library call in an `unsafe` block as our
wrappings can handle this and only expose the aspects we want. Further it really
improved the notion of constants, all the `#def` within C library can now be
represented using enums and hence better express their purpose and context for
usage.

### Mapping Constants

The typical way to represent constants within C would be like the following:
```
#def LOW 0
#def HIGH 1
```
This is very common for a language like C but we can do a lot better in Rust to make it far simpler to separate concepts. The issue that constants like these raise is that we end up with functions that accept integers for which we just pass some value in that was included from some distant header file.

What would be far clearer is representing this as an enumeration . The Rust representation for enumerations is very powerful but it doesn't have the concept of ordinals, so I found the simplest way was to have a trait that exposed this, with the eventual hope to make a macro of it.
```
enum DigitalValue {
    Low = 0,
    High = 1,
}
```

Which allows us to be explicit in the types we can give to functions using these constants:
```
fn digital_write(value: DigitalValue)
```

And access them as:
```
bindings::digitalWrite(value as i32);
```

## Cross compilation for Raspberry Pi

The general idea of cross compilation I have already covered in a [previous
post][cross_comp] so I won't go into that here but just the specifics required for this
project.

This part of the project I managed to get most of my answers from the Rust
community from an [SO answer] to the original [rust-wiringpi]. The former in
what was the correct target for the Raspberry Pi and the latter for how to build
the WiringPi library as part of the bindings build, hence ensuring the build
library would have the correct architecture. Put simply if you follow the general instructions I provided in the [Cross Compilation][cross_comp] post the target you are wanting is: **arm-unknown-linux-gnueabihf**.


The two aspects go hand in hand as essentially what we are doing is ensuring we
have the linker for the correct target and configuring first Cargo to do this
correctly for us, then secondly we use the build tools of the library we are
binding to, to ensure that the linker used there is that same as the target we
are going for. In order to get the WiringPi library built for the target I followed off what was done in the original bindings which was to add a Makefile for which we set the `CC` variable to the linker that we were requiring for the Raspberry Pi.

```
CC=arm-linux-gnueabihf-gcc
```

After adding this I was able to overcome all of the build issues and then with a simple `scp` moved the built binary which you can find in `target/arm-unknown-linux-gnueabihf/debug/` or `target/arm-unknown-linux-gnueabihf/release` the LED was blinking from the Raspberry Pi!

[guide]: https://servo.github.io/rust-bindgen/
[wiringPi library]: http://wiringpi.com
[rust-wiringpi]: https://github.com/Ogeon/rust-wiringpi
[git submodule]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
[cross_comp]: cross_compilation.html
[SO answer]:https://stackoverflow.com/questions/29917513/how-can-i-compile-rust-code-to-run-on-a-raspberry-pi-2
[repo]:https://github.com/maccoda/wiringpi-rs
