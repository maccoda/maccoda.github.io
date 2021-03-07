# Maccoda Mail

This is the content of my blog site.

It uses [zola](https://www.getzola.org/) to generate the content, which needs to
be installed as a separate binary, it is not part of this repository.

## Branching structure

I do not exactly remember as to why but it was something to do with Github pages
that required all of my content creation to be on the `writing` branch so that
the published version is on the `master` branch.

## Content Creation

### Diagrams

I have hooked [mermaid](http://mermaid-js.github.io/mermaid/#/) into the build
cycle, therefore any code snippet of type `mermaid` will result in a rendered
diagram.

## Usage

```shell
> zola serve
```
