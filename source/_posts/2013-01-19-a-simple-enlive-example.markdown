---
layout: post
title: "A simple Enlive example"
date: 2013-01-19 22:17
comments: true
categories:
published: false
---

[Enlive](https://github.com/cgrand/enlive) is a selector-based
templating system. Rather than templates containing code which is
executed to produce the final output, they are plain HTML which is
transformed by code which hooks into them using a version of standard
CSS selectors. This approach allows a clean separation between
template and code; it avoids creating a novel, often crippled and
always nasty hybrid language for the templates.

As you can tell, I am pretty sold on the benefits of selector-based
templating, so I was excited to come across Enlive when I started
playing with Clojure. Enlive is powerful and somewhat complex; there
are two [good] [marick] [tutorials] [nolen] but I found it hard to get
straight in my head which bits were really necessary to do something
simple.

[marick]: https://github.com/cgrand/enlive/wiki/Table-and-Layout-Tutorial,-Part-1:-The-Goal "Brian Marick's tutorial"
[nolen]: https://github.com/swannodette/enlive-tutorial/ "David Nolen's tutorial"

So, having wrestled with this for a while (and having spent some time
reading the Enlive source code), I've put together a [basic example]
[github-project], which I think takes a reasonable, minimal approach
to using Enlive for templating a simple web application.

[github-project]: https://github.com/benbc/simple-enlive-example
