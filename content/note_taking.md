+++
date = 2018-04-04
title = "Note Taking in Visual Studio Code"
description = "How you can set up VSCode to be a useful note taking application rather than getting another application just dedicated for notes."
[taxonomies]
tags = ["notes"]
+++

tl;dr Installed some extensions and made some snippets to make a really nice
note taking experience. Exact changes needed are [below](#setting-it-all-up).

## Visual Studio Code

First and foremost I would like to introduce [Visual Studio Code] if you haven't
heard of it yet. It is my editor of choice and is very flexible and extendable.
For more information on it check out their site, I definitely recommend having a
look!

[Visual Studio Code]: https://code.visualstudio.com

Now that you have VSCode I want to share the set of extensions, snippets, etc.
that I put together to create what I believe to be a very useful setup to allow
you to use VSCode for note taking. In particular I prefer to take my notes in
Markdown, if this isn't your preferred I hope I can still assist in some way.

One thing I have noticed is that my laptop consistently starts getting filled up
with more and more applications during my work with software. An IDE (usually a
few even), an editor, database visualizer, email client, internet browsers
(multiple), etc. The last thing I want to do is add another application where I
could use an existing one in its place, this is what triggered me to look into
what alternatives were out there. Considering I am pretty mainstream with my
Markdown note taking it wasn't too much of a difficult task. I already used
VSCode for many of my projects and plenty for editing READMEs and the like, so I
knew that it had quite an expansive set of functionality baked in to provide a
great experience when writing Markdown.

## Dedicated Note Taking Applications

Before I started this I definitely had a look at existing applications because,
hey who doesn't want to just take something off the shelf and it just works.
Again I specifically looked at the areas around Markdown based note taking and I
noted a few things that I found to be the most useful.

1. Instant preview - a lot of these was in a side by side view
1. Tags - This is a great feature and easy way of searching and grouping notes
1. 'Notebooks' - Being able to separate the notes into 'notebooks' is a very
   logical thing to be able to group notes together for easy reference.
1. Direct access - not so much a functionality of the application itself but I
   enjoy being able to have direct access to my notes and be able to treat them
   as their own separate entity.

After some tinkering and searching I was very excited to find that I could
create a very similar experience in my already existing VSCode.

## Setting it all up

### VSNotes

Extensions are the name given to plugins for VSCode. They are easily installed
from the extension manager in the sidebar, or even through the command palette
directly!

The first extension you will want is [VSNotes]. This provides the majority of
the functionality. It provides:

- Direct access to your notes through the _Explorer_ side bar or _Command
  Palette_
- Tags using the Markdown front matter of your notes

#### Settings

All that you need to do to make the most of this change the setting for the home
directory of your notes:

```json
"vsnotes.defaultNotePath": "/path/to/notes"
```

#### Snippets

Then create the following snippet for Markdown files. This snippet can be
created by accessing the _Command Palette_ then selecting `Preferences:
Configure User Snippets` > `markdown`. This will allow you to create language
specific snippets for Markdown.

```json
"Front Matter Tags": {
		"prefix": "tags",
		"body": [
			"---",
			"tags:",
			"    - $1",
			"---"
		]
	}
```

### Auto Preview

The final extension was [Auto-Open Markdown Preview][auto-preview] which simply
does exactly as the name suggests. When a Markdown file is opened it
automatically opens the side preview of the Markdown file (which I usually can
open from `ctrl + shift + m`).

### Notebooks

The last part is the least inventive of how to address the final point of
notebooks. I am simply using directory structure to solve this problem. However
I am finding that this is sufficient thanks to the structure preview provided by
**VSNotes**. I know I am sorry a not so exciting note to end on but a simple
one.

[VSNotes]: https://marketplace.visualstudio.com/items?itemName=patricklee.vsnotes
[auto-preview]: https://marketplace.visualstudio.com/items?itemName=hnw.vscode-auto-open-markdown-preview

## Conclusion

All in all I am finding this to be a suitable solution for my note taking
problem and hope this helps someone. I am also keen to hear if anyone else has a
better way of doing this or the like. Probably best way to discuss is to raise
an issue on the [repo](https://github.com/maccoda/maccoda.github.io/issues).
