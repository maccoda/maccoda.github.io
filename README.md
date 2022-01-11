# Maccoda Mail

This is the content of my blog site.

It uses [zola](https://www.getzola.org/) to generate the content, which needs to
be installed as a separate binary, it is not part of this repository.

The theme is connected via a submodule, therefore run the following to fetch it

```shell
> git submodule update --init --recursive
```

## Content Creation

### Diagrams

I have hooked [mermaid](http://mermaid-js.github.io/mermaid/#/) into the build
cycle, therefore any code snippet of type `mermaid` will result in a rendered
diagram.

## Usage

```shell
> zola serve
```
