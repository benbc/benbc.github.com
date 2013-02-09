---
layout: post
title: "A thin layer over Enlive"
date: 2013-02-09 19:10
comments: true
categories:
---

In my [last post] [simple example] I showed a simple use of Enlive to
create a web application with a common layout for all pages. I
resisted the temptation to introduce any abstractions because I wanted
to make it absolutely clear how to use the building blocks that Enlive
provides.

But my fingers were itching the whole time to abstract away some of
the wrinkles and I couldn't let it rest until I'd had a play to see
what it looks like. So here is a very thin layer over Enlive that
manifests some of the structure that I saw.

<!--more-->

Here's the code we ended up with last time. There is a `layout`
template that is used to wrap the two pages (`index` and `show`).

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

(defroutes app
  (GET "/" [] (layout "Front page" index))
  (GET "/show" [] (layout "Show things" (show things))))
```

I'd like to find a way to clarify the *page* and *layout* concepts in
the code. I'll use macros to define them, so that I get the
define-and-assign style effect which I think is appropriate for
important entities like this.

Layouts first. There is some slightly tricky stuff here, stripping off
unwanted `<html>` and `<body>` tags, which we can hide. Let's assume
that every layout will have one main piece of content, which we can
define by convention will go into `div.content`; but we'll allow each
template to define other slots and substitutions for them.

So we want the layout definition for our example above to look like
this.

``` clojure
(deflayout layout "layout.html" [title]
  [#{:title :h1}] (enlive/content title))
```

This defines a function (`layout`) which takes a single argument
(`title`) and returns an Enlive *template*&mdash;another function that
takes the page's content and returns the rendered HTML. We're
effectively currying the `layout` function defined in the raw example
because we want to specify `title` and `content` at different points.

Here is how we would use it, without changing anything else.

``` clojure
(defroutes app
  (GET "/" [] ((layout "Front page") index))
  (GET "/show" [] ((layout "Show things") (show things))))
```

And here's what `deflayout` looks like.

``` clojure
(defmacro deflayout [name source args & forms]
  `(defn ~name ~args
     (enlive/template ~source [content#]
                      [:div.content] (enlive/substitute (extract-body content#))
                      ~@forms)))
```

And now pages. There are two different kinds of page: dynamic ones,
like `show`, which take one or more parameters and transform their
HTML and static ones, like `index`, with neither parameters nor
transformations. We'd like to make defining and calling both types
simple. We'd also like to specify the layout when defining the page
and hide its application. Here's what we're aiming for.

``` clojure
(defpage show "show.html" (layout "Show things") [things]
  [:li] (enlive/clone-for [thing things] (enlive/content thing)))

(defpage index "index.html" (layout "Front page"))
```

And here's the definition of `defpage`.

``` clojure
(defmacro defpage
  ([name source layout]
     `(def ~name
        (~layout (enlive/html-resource ~source))))
  ([name source layout args & forms]
     `(defn ~name ~args
        (~layout (enlive/at (enlive/html-resource ~source)
                            ~@forms)))))
```

This creates functions for parameterized pages and simple values for
static ones to simplify calling them.

``` clojure
(defroutes app
  (GET "/" [] index)
  (GET "/show" [] (show things))
  (not-found "Not Found"))
```

You can see the complete code, in a working project, [here] [github].

This refactoring is obviously overkill for a tiny example like this,
but something like it could be useful on a real project. It meets the
two objectives, anyway: clarifying the important concepts and hiding
some of the fiddly details.

[simple example]: /blog/2013/01/27/a-simple-enlive-example/
[github]: https://github.com/benbc/simple-enlive-example/blob/abstractions/src/simple_enlive_example/main.clj
