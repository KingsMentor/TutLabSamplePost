![Showdown][sd-logo]

Showdown is a Javascript Markdown to HTML converter, based on the original works by John Gruber. It can be used client side (in the browser) or server side (with Node or io). 


# Installation

## Download tarball

You can download the latest release tarball directly from [releases][releases]

## Bower

    bower install showdown

## npm (server-side)

    npm install showdown

## CDN

You can also use one of several CDNs available: 

* rawgit CDN

        https://cdn.rawgit.com/showdownjs/showdown/<version tag>/dist/showdown.min.js

* cdnjs

        https://cdnjs.cloudflare.com/ajax/libs/showdown/<version tag>/showdown.min.js


[sd-logo]: https://raw.githubusercontent.com/showdownjs/logo/master/dist/logo.readme.png
[releases]: https://github.com/showdownjs/showdown/releases
[atx]: http://www.aaronsw.com/2002/atx/intro
[setext]: https://en.wikipedia.org/wiki/Setext

---------

# Syntax


## Introduction

Showdown was created by John Fraser as a direct port of the original parser written by markdown's creator, John Gruber. Although Showdown has evolved since its inception, in "vanilla mode", it tries to follow the [original markdown spec][md-spec] (henceforth refereed as vanilla) as much as possible. There are, however, a few important differences, mainly due to inconsistencies in the original spec, which we addressed following the author's advice as stated in the [markdown's "official" newsletter][md-newsletter].

Showdown also support "extra" syntax not defined in the original spec as opt-in features. This means new syntax elements are not enabled by default and require users to enable them through options.

This document provides a quick description the syntax supported and the differences in output from the original markdown.pl implementation.

## Paragraphs

Paragraphs in Showdown are just **one or more lines of consecutive text** followed by one or more blank lines.

```md
On July 2, an alien mothership entered Earth's orbit and deployed several dozen 
saucer-shaped "destroyer" spacecraft, each 15 miles (24 km) wide.
    
On July 3, the Black Knights, a squadron of Marine Corps F/A-18 Hornets, 
participated in an assault on a destroyer near the city of Los Angeles.
```

The implication of the “one or more consecutive lines of text” is that Showdown supports 
“hard-wrapped” text paragraphs. This means the following examples produce the same output:

```md
A very long line of text
```

```md
A very
long line
of text
```
