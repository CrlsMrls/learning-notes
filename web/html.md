# HTML

## HTML elements terminology

### Anatomy of an HTML element

- The opening tag
- The closing tag
- The content
- Attributes contain extra information about the element, e.g. `class`
- The element: All previous together.

### Empty elements
Empty elements consist only of a single tag, e.g `<img>`.

### Block versus inline elements

Block-level elements form a visible block on a page — they will appear on a new line.
Inline elements are those that are contained within block-level elements. It will not cause a new line to appear.

HTML5 redefined the element categories in HTML5: see [Element content categories](http://www.whatwg.org/specs/web-apps/current-work/complete/section-index.html#element-content-categories).

## Head element

The head's job is to contain metadata about the document.

- Specifying your document's character encoding: `<meta charset="utf-8">`
- Adding an author and description: `<meta name="author" content="Chris Mills">` or `<meta name="description" content="The MDN Web...">`

## HTML semantics

We rely on previous experience to tell us what the function of everyday objects is; when we see something, we know what its function will be.

HTML5 redefined `<b>`, `<i>` and `<u>` with new semantic roles. These old elements only affect presentation and not semantics, are known as presentational elements and should no longer be used. Instead use the following:

### Emphasis

Stress certain words, subtly altering the meaning of what we are saying. `<em></em>`

### Strong importance

To emphasize important words: `<strong></strong>`

### Representing computer code

`<code>` For marking up generic pieces of computer code.
`<pre>` For retaining whitespace (generally code blocks)
`<var>` For specifically marking up variable names.
`<kbd>` For marking up keyboard.
`<samp>` For marking up the output of a computer program.

### Marking up times and dates

`<time>` marks up times and dates in a machine-readable format: `<time datetime="2016-01-20">20 January 2016</time>`

### Structure

`<main>` Use it only once per page, and put it directly inside <body>.
`<article>` encloses a block of related content that makes sense on its own without the rest of the page
`<section>` is similar to <article>, but it is more for grouping together a single part of the page
`<aside>` not directly related to the main content but additional information (e.g glossary entries, author biography, related links, etc.)
`<header>` represents a group of introductory content. If it's a child of an `<article>` or `<section>` it defines a specific header for that section
`<nav>` contains the **main navigation** functionality for the page.
`<footer>` represents a group of end content for a page.

### Non-semantic wrappers

When there is not an ideal semantic element to group some items together, you can use `<div>` or `<span>`. Only use them if there is no better semantic element:
`<span>` is an inline non-semantic element.
`<div>` is a block level non-semantic element.

## Images

If an image has meaning, in terms of your content, you should use an HTML image. If an image is purely decoration, you should use CSS background images.

The <figcaption> element tells browsers, and assistive technology that the caption describes the other content of the <figure> element. It semantically links the image to its caption.

```html
<figure>
  <img
    src="images/dinosaur.jpg"
    alt="The head and torso of a dinosaur skeleton"
    width="400"
    height="341"
  />
  <figcaption>A T-Rex.</figcaption>
</figure>
```

`title` has a number of accessibility problems, ... most browsers won't show it unless you are hovering with a mouse
