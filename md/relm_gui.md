# Simple GUI in Relm

This work stemmed from the article written on [A Simple Rust GUI with
QML][qml-article] and reused a bit of the back end there.

## What is Relm
Recently the GUI library [Relm][relm] came onto the scene with the goal to develop asynchronous GUI applications in Rust. The link provides a great description as to it benefits that it brings so I won't bore you with the details a second time around.
In short what really sold me initially (as someone quite unfamiliar with GUI work) was:
* The ability to have a declarative nature in which you could define your view
* The simplicity with dealing with a model
* The simplicity of defining the events or messages received

Of this the biggest winner for me was the abstraction of the model manipulation
as I wanted to try a few different frameworks so I could have a go to when
writing GUI programs and the difficultly was handling the mutability of your
model. This is notion is put in the limelight with Rust due to its borrow
checker. However I do want to point out that this is by no means a shortcoming
of Rust but rather highlights one of the more common issues in UI and all
development is a mutable state which is able to altered from several locations. Anyway Rust fanfare aside, Relm has done a great job in wrapping this complexity to make a very simple library to work with I believe.

## Getting our hands dirty
Relm is quite well documented and has several examples that were easy to follow so again I won't repeat these but it was very easy to get a basic example of a button that is able to report when it is clicked an increment a counter.

One thing I do want to add before we go deeper is that Relm is strongly related to GTK concepts, in particular the widget elements used are GTK widgets. So I would really recommend having at least a reference to it open when working through it if you aren't familiar with the nomenclature of the widgets.

So my test was to see how simple it would be to get the basic file explorer example shown in [the QML article][qml-article] using Relm. To my surprise I managed to get it done in under 200 line (199 to be exact XD). If all you want is to have a look at the code and play around with it (be warned I wouldn't say it is beautiful just yet) I have made a [gist][gist], otherwise if you'd like to stick around we can build it up bit by bit.

## TO BE CONTINUED...

[qml-article]: https://www.vandenoever.info/blog/2017/02/17/a-simple-rust-gui-with-qml.html
[relm]: http://relm.ml/relm-intro
[gist]: https://gist.github.com/maccoda/a93fb2a0ef1f283bc63a4be9e5f8fe9c
