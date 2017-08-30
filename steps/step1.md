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

    x = 2 + 2
    what is x
    ```

## Lists

Showdown supports ordered (numbered) and unordered (bulleted) lists.

### Unordered lists

You can make an unordered list by preceding list items with either a *, a - or a +. Markers are interchangeable too.

```md
* Item
+ Item
- Item
```

### Ordered lists

You can make an ordered list by preceding list items with a number.

```md
1. Item 1
2. Item 2
3. Item 3
```

It’s important to note that the actual numbers you use to mark the list have no effect on the HTML output Showdown produces. So you can use the same number in all items if you wish to.

### TaskLists (GFM Style)

Showdown also supports GFM styled takslists if the **`tasklists`** option is enabled.

```md
 - [x] checked list item
 - [ ] unchecked list item
``` 

 - [x] checked list item
 - [ ] unchecked list item

### List syntax

List markers typically start at the left margin, but may be indented by up to three spaces. 

```md
   * this is valid
   * this is too  
```

List markers must be followed by one or more spaces or a tab.

To make lists look nice, you can wrap items with hanging indents:

```md
*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.
```

But if you want to be lazy, you don’t have to

If one list item is separated by a blank line, Showdown will wrap all the list items in `<p>` tags in the HTML output.
So this input:

```md
* Bird

* Magic
* Johnson
```
