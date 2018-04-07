+++
date = 2017-05-12
title = "Simple GUI in Relm"
description = "Relm is a GUI framework built off GTK and looked like a great candidate for my first Rust GUI program."
+++
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

## Model
First things first, we want to define our model of our little application. For those note so familiar with this concept the model is data that will adequately explain the state of your application. Since it is just a very simple file browser all we need is the current directory. So it will look a little something like this:
```rust
#[derive(Clone)]
pub struct Directory {
    current_dir: PathBuf,
}
```
Note that you will need your model to implement the `Clone` trait to used as part of Relm.

## Message
Now we don't just a static window that doesn't respond to much so we need a way of sending events/messages to update the state of our application. The beauty of Relm is just how simple it makes it to handle several events and even more so you are able to categorize the events logically in terms of your application as opposed to the underlying framework. That is, if you have several different UI events that from the perspective of your application are indifferent then you are able to send the same message for both with a more descriptive name.

Now this example doesn't really delve into that but it isn't too much of a stretch to see how it could be great. For our little application we only care when the user selects an item in the list of files we present to them and of course when they want to quit.

```rust
#[derive(Msg)]
enum Msg {
    ItemSelect,
    Quit,
}
```

Here we use the one of the biggest time savers in Rust, the `derive` attribute to derive the `Msg` trait from Relm.

## Creating the Main Widget
The final step now is to create our widget. As seen in the [intro to Relm][relm] there are two methods to this using the `#[widget]` attribute or implementing the `Widget` trait. WIth the structure of this widget I had a bit of difficulty getting the `#[widget]` attribute to entirely agree with me so I implemented the trait which I did not mind at all as I always feel that the amazing macro system in Rust can add a layer of complexity when it comes to solving errors.

Below is the full implementation for which we will look at individually.
```rust
// Top level structure
#[derive(Clone)]
struct Win {
    window: gtk::Window,
    tree_view: gtk::TreeView,
}

impl Widget for Win {
    type Model = Directory;
    type Msg = Msg;
    type Root = gtk::Window;

    fn model() -> Directory {
        let working_directory = fs::canonicalize(".").expect("Failed to open directory");

        Directory { current_dir: working_directory }
    }

    fn root(&self) -> &Self::Root {
        &self.window
    }

    fn update(&mut self, event: Msg, model: &mut Self::Model) {
        match event {
            Msg::ItemSelect => {
                let selection = self.tree_view.get_selection();
                if let Some((list_model, iter)) = selection.get_selected() {
                    let is_dir: bool = list_model
                        .get_value(&iter, IS_DIR_COL)
                        .get::<bool>()
                        .unwrap();

                    if is_dir {
                        let dir_name = list_model
                            .get_value(&iter, VALUE_COL)
                            .get::<String>()
                            .unwrap();
                        println!("{:?} selected", dir_name);
                        let new_dir = if dir_name == ".." {
                            // Go up parent directory if it exists
                            model
                                .current_dir
                                .parent()
                                .unwrap_or(&model.current_dir)
                                .to_owned()
                        } else {
                            model.current_dir.join(dir_name)
                        };

                        model.current_dir = new_dir;
                        let new_model = create_and_fill_model(&model.current_dir).unwrap();

                        self.tree_view.set_model(Some(&new_model));
                    }
                }
            }
            Msg::Quit => gtk::main_quit(),
        }
    }

    fn view(relm: RemoteRelm<Msg>, _model: &Self::Model) -> Win {
        let window = create_window("Treeview");

        let scroll = gtk::ScrolledWindow::new(None, None);
        let tree = gtk::TreeView::new();


        let column = gtk::TreeViewColumn::new();
        let cell = gtk::CellRendererText::new();

        column.pack_start(&cell, true);
        // Association of the view's column with the model's 'id' column.
        let id = 0;
        column.add_attribute(&cell, "text", id);
        tree.append_column(&column);


        let model = create_and_fill_model(&_model.current_dir).unwrap();

        tree.set_model(Some(&model));

        // tree.connect_cursor_changed(change_dir);
        connect!(relm, tree, connect_cursor_changed(_), Msg::ItemSelect);
        connect!(relm, window, connect_delete_event(_, _) (Some(Msg::Quit), Inhibit(false)));

        scroll.add(&tree);
        window.add(&scroll);

        window.show_all();

        Win {
            window: window,
            tree_view: tree,
        }
    }
}
```

### Top Level Widget
```rust
struct Win {
    window: gtk::Window,
    tree_view: gtk::TreeView,
}
```
Of course we have to have a root or top level widget, the one that will contain all other widgets. This is exactly what the `Win` struct is, it is our window (mind the pun) into the GTK widgets. Here we have the encasing `gtk::window` for our root widget and we wanted to open access to the `gtk::TreeView` which will contain the file listing.

### Widget Type Declarations
When implementing the `Widget` trait we need to declare
* the `Model` of our application already discussed in the [model](#model) section,
* the `Msg` being the events to communicate changes already discussed in the [message](#message) section,
* the `Root` element of the application, this is the GTK element that will house the other widgets. For us this is the `gtk::Window`.

For the notions of `Model` and `Root`, Relm's `Widget` provides a function to construct/access these through the `model()` function to initialize the model, and `root()` to provide access to the root element.

### View
Now we finally have to build our view to describe how it should look. This is where Relm made the delightful `view!` macro to allow for a view to be built in a declarative fashion. However I found that it was a bit easier to do away with this and just simply implement the trait function in the usual fashion. This was partly due to the fact that I had already played around with this idea with GTK and had some code at hand.

The goal of the `view()` function is to construct the view exactly as desired and return our [top level widget](#top-level-widget). In here also we add all of event listeners using the simple `connect!` macro.
```rust
connect!(relm, tree, connect_cursor_changed(_), Msg::ItemSelect);
        connect!(relm, window, connect_delete_event(_, _) (Some(Msg::Quit), Inhibit(false)));
```

Don't forget to make it visible also!
```rust
window.show_all();
```

### Updating From Events
The final part of this trait to implement is the `update()` function. This is where our events will be handled depending on our [message](#message) received. Since there isn't a lot going on here I haven't gone for the asynchronous version of this and the function signature is the following:
```rust
fn update(&mut self, event: Msg, model: &mut Self::Model)
```

As you can see from this function we have:
* `&mut self` so we are able to manipulate the main widget, which in our case will be with the intention to update the `gtk::TreeView` when a new directory is selected.
* `event: Msg` so that we can perform a simple `match` on the event received and have clear separation of our event handling code.
* `&mut Self::Model` which will provide mutable access to the model, allowing us to update it if required from the event.

### But what about the other code?
As you may notice in the gist there is a whole bunch of other code that I had there more related to the construction of the GTK elements. This is heavily based on example code from `gtk-rs` which now has even more documentation so thought could just leave that for the reader to understand XD.

## Actually Running It
Of course what good is all this if we cannot run it! Just make sure your main looks like the following and it all can get happening!
```rust
fn main() {
    relm::run::<Win>().unwrap();
}
```

# Conclusion
Overall I found Relm as a great addition to the Rust GUI libraries and very simple to use especially when UI is not your strong suit. It abstracts away several repeated details that we required to make the borrow checker smile allowing the user to put something together quite quickly.

That's all for the first post. Hope this has helped!

[qml-article]: https://www.vandenoever.info/blog/2017/02/17/a-simple-rust-gui-with-qml.html
[relm]: http://relm.ml/relm-intro
[gist]: https://gist.github.com/maccoda/a93fb2a0ef1f283bc63a4be9e5f8fe9c
