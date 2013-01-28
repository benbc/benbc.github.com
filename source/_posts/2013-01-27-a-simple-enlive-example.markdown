---
layout: post
title: "A simple Enlive example"
date: 2013-01-27 22:17
comments: true
categories:
published: false
---

[Enlive] [enlive] is a Clojure library for generating HTML that uses
_transformations_ instead of _templates_. Rather than starting with
templates containing code which are then executed to produce the final
output, it starts with plain HTML which is subjected to a series of
transformations; the transformations are ordinary functions, targeted
to the right part of the DOM by standard CSS selectors.

This approach allows a clean separation between template and code; it
avoids creating a novel, often crippled and always nasty hybrid
language for the templates.

As you can tell, I am pretty sold on the benefits of selector-based
templating, so I was excited to come across Enlive when I started
playing with Clojure. Enlive is powerful and somewhat complex; there
are two [good] [marick] [tutorials] [nolen] but I found it hard to get
straight in my head which bits were really necessary to do something
simple.

[marick]: https://github.com/cgrand/enlive/wiki/Table-and-Layout-Tutorial,-Part-1:-The-Goal "Brian Marick's tutorial"
[nolen]: https://github.com/swannodette/enlive-tutorial/ "David Nolen's tutorial"

So, having wrestled with this for a while (and having surface after a
longish dive in the Enlive source code), I've put together a [basic
example] [github-project], which I think takes a reasonable, minimal
approach to using Enlive for a simple web application. It is not
intended to be a complete introduction to what Enlive can do; for
that, read the tutorials and [documentation] [enlive].

When using Enlive, it's helpful to know that the library generally
uses an internal data structure representation of HTML (called *nodes*
in the documentation). Most functions and macros operate on and return
*nodes*, or collections of them; a few return strings, which you need
in order to render the HTML to the outside world (in my case when
defining Compojure routes).

The simplest thing we can do is read in an HTML file from disk with
`html-resource`.

``` clojure
(def index (html-resource "index.html"))
```

This returns *nodes*, which can be rendered into strings with `emit*`.

``` clojure
(defroutes routes
  (GET "/" [] (emit* index)))
```

We would also like to be able to show dynamic content. With a
transformation-based approach, we start with static HTML and apply
transformations to it. We're going to create a `<ul>`, with one `<li>`
for each item in a list. The initial HTML shows the structure we want,
with dummy content in the `<li>`.

``` html
<ul>
  <li><!-- item --></li>
</ul>
```

Again we read in the HTML using `html-resource`, but this time we use
`at` to apply a transformation.

``` clojure
(defn show [things]
  (at (html-resource "show.html")
      [:li] (clone-for [thing things] (content thing))))
```

This targets the `<li>` element and applies a transformation which,
for every element in the argument list `things`, clones the `<li>` and
fills in its content.

Like `html-resource`, `at` returns *nodes*, so we can render the
result with `emit*`.

``` clojure
(def things ["one" "two" "three" "four"])

(defroutes routes
  (GET "/" [] (emit* index))
  (GET "/show" [] (emit* (show things))))
```

For a real web application, we need a way of applying common layout
and styling to the pages. With Enlive we can use a further
transformation to insert the pages that we have so far into a common piece of wrapper HTML.

``` html
<html>
  <head>
    <style type="text/css">
      body { font-family: sans-serif }
    </style>
  </head>
  <body>
    <div class="content">
      Content goes here.
    </div>
  </body>
</html>
```

Enlive provides a macro `deftemplate` that we can use here. It wraps a
couple of the things we've seen already (`at` and `html-resource`) as
well as doing the function definition for us.

``` clojure
(deftemplate layout "layout.html" [content]
  [:div.content] (substitute content))
```

This replaces the contents of `div.content` with the HTML that we pass
it. `deftemplate` also handles the call to `emit*` for us, so replace
it with `layout` when defining our routes.

``` clojure
(defroutes routes
  (GET "/" [] (layout index))
  (GET "/show" [] (layout (show things))))
```

We can improve `layout` to apply consistent titles to each page, by
adding an argument and setting the content of `<title>` and `<h1>`
tags.

``` clojure
(deftemplate layout "layout.html" [title content]
  [#{:title :h1}] (content title)
  [:div.content] (substitute content))

(defroutes routes
  (GET "/" [] (layout "Front page" index))
  (GET "/show" [] (layout "Show things" (show things))))
```

And we add those elements, with placeholders, to the layout HTML.

``` html
<html>
  <head>
    <title>Title goes here</title>
    <style type="text/css">
      body { font-family: sans-serif }
    </style>
  </head>
  <body>
    <h1>Title goes here</h1>
    <div class="content">
      Content goes here.
    </div>
  </body>
</html>
```

There is one wrinkle that I've glossed over up to this point. Enlive
uses the wonderful [TagSoup] [tagsoup] to parse HTML (so that you can
use the same transformations on "wild" HTML that you've scraped from
the web and which may not be well-formed). Because TagSoup assumes
that it's dealing with something potentially toxic, and tries to
detoxify it, it adds `<head>` and `<body>` tags around any HTML that
you give it. So the DOM returned by `html-resource` has these extra
tags which end up wrapping the content in the middle of our webpage.
We need to modify `layout` to strip these out.

[tagsoup]: http://ccil.org/~cowan/XML/tagsoup/

Here is the complete code, which you can see in a working project
[here] [github-project].

``` clojure
(def things ["one" "two" "three" "four"])

(defn extract-body [html]
  (at html [#{:html :body}] unwrap))

(deftemplate layout "layout.html" [title content]
  [#{:title :h1}] (content title)
  [:div.content] (substitute (extract-body content)))

(defn show [things]
  (at (html-resource "show.html")
             [:li] (clone-for [thing things] (content thing))))

(def index (html-resource "index.html"))

(defroutes routes
  (GET "/" [] (layout "Front page" index))
  (GET "/show" [] (layout "Show things" (show things))))
```

[github-project]: https://github.com/benbc/simple-enlive-example
[enlive]: https://github.com/cgrand/enlive
