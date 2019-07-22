+++
date = 2017-05-13
title = "Cross Compilation for Rust (Windows on Linux)"
description = "Looking into how to compile Windows binaries on a Linux machine for Rust."
[taxonomies]
tags = ["rust", "cross compilation"]
+++
Cross compilation always has an aura about it, being able to develop on one
machine and be able to produce executables for all devices. I mean that was
Java's selling point "Write it once, run it everywhere" (I probably got that one
wrong but you get the point). This was one hurdle for me with Rust as I use a
Linux desktop at work but I was to be able to make things that I can give family
and friends so that they may benefit from it also. In particular a lot of
corporate positions use Windows (myself included) meaning it is difficult to get
my executables to work. However after a bit of reading and in particular the
[Cross Repository][cross] I have managed to get cross compilation for Linux
(Ubuntu) to Windows working and thought I would send this one out :).

## Building with Cargo
For those not so familiar with Rust, Cargo is the build tool actually shipped
with Rust. I must say I find this to be a very well planned and built build
tool. It handle a lot of the common issues with the build process and makes it
extremely simple to get applications built and running quickly. Essentially it
add a lot more brain power to the `rustc` compiler, particularly in the
department of dependency management and tasks (build, test, etc.).

Where it comes in even more useful here is that when I want to build for a
different target there is not much more work than a few commands and then simply
add the `--target=<target>` argument and viola I have my cross compilation!

## What is a Target?
Put simply the target is the target architecture that your executable is going to be running on. It is fairly common knowledge that my executable on a Linux machine can't run exactly on a Windows, and only sometimes on a Mac (Don't quote me on that last one). Again this is very well detailed in the [Cross Repository][cross] what Rust defines as targets but will have a quick summary here.

Essentially it boils down to a few things. First and foremost the
**architecture**. This is the architecture of the actual processor that is
running your software. Is it a 32-bit or 64-bit machine? Or is it perhaps for an
embedded platform and running on an ARM processor? Excitingly, all of these are
possible targets for Rust!

The other main aspect is the **system**, in here we separate Linux from Mac from Windows. Of course there are may others than just these ones but these are your main players. From these and some other characteristics we create a **triple** that explicitly defines a target.

## Cross-Compiling
Alight enough background, let's get into what needs to be done to cross compile.

*Now little disclaimer, I have only done this for my machine compiling for Windows on my Linux machine. I have not tried to the converse or any other combination.*

First off we need to add the target to your machine, in particular this adds the standard library for the intended target. Again thanks to the incredible tooling from the Rust community this is simple using `rustup`.

```
$ rustup target add x86_64-pc-windows-gnu
```

Next we need to get the `gcc` compiler for that target. If you are interested I found it fascinating the amount of targets gcc itself has as I looked through which was the correct package to get. Using a Debian system:
```
$ sudo apt-get install gcc-mingw-w64
```
You should then find in `/usr/lib/gcc` all your `gcc` targets.
```
[dmaccora:/usr/lib/gcc] $ ls
i686-w64-mingw32  x86_64-linux-gnu  x86_64-w64-mingw32
```

Next we need to change the Cargo configuration so that it uses the newly
installed `gcc` linker instead of the system one. Of course we only want to use
this for a particular target but of course Cargo has got this covered for us
with its [configuration options][cargo-config]. So it applies for all projects
we place the configuration in `~/.cargo/config`. Don't worry if this file
doesn't exist just create it new, Cargo has a default configuration it will be
using unless otherwise specified (which is what we will do now). All that is needed to add the following two lines:
```
[target.x86_64-pc-windows-gnu]
linker = "x86_64-w64-mingw32-gcc"
```
You will note that the linker matches the `gcc` target we found earlier (hopefully this is the general rule as that is how I managed to get it all working).


Then finally we can build our crate for another target!
```
$ cargo build --target=x86_64-pc-windows-gnu
```



[cross]: https://github.com/japaric/rust-cross
[cargo-config]: http://doc.crates.io/config.html
