+++
title = "New website! (finally)"
description = "Ramblings about recreating my personal website"
tags = ["personal"]
categories = ["Personal"]
# series = []
date = 2019-05-10T09:59:40-07:00
+++

This website has taken much longer to get online than it should have.
As is usually the case, I couldn't decide what I wanted the new site to look like,
and in the end I decided on the most simple design possible (as you can see).

The lesson to be learned here is probably not to waste too much time on your personal website.
Unless you're a designer or artist, the main activity users are doing on a personal site
isi probably reading, so keeping it clean and readable should be the top priority.

Despite the timesink, I did learn about a few new tools on the way.

First I'd like to mention that the CSS and JS libraries used on this site were from looking at
[hugo-theme-basic](https://github.com/siegerts/hugo-theme-basic) made by Stephen Siegert,
so thanks for making that theme.

[Tachyons](http://tachyons.io/) is a really neat CSS framework. It's one file,
and the styles are all short names that can quickly be applied to the "class"
attribute of any HTML element. I was originally deciding between writing my own CSS and
using Bootstrap (which is overkill for such a simple website), but I think Tachyons is
a good middle ground when you want to save time but don't need all the extras that come
with a larger CSS framework.

I have used [Hugo](https://gohugo.io/) before, and this is the second website I've
created with it. I would still strongly recommend it to anyone looking to make a
website without server side logic. Working with static site generators feels much
more like developing for the old web (the nice parts of it anyway), but with the
added benefit of modern HTML, CSS, and Javascript.

I don't work much with Javascript anymore, but [highlight.js](https://highlightjs.org/)
is one of the easiest syntax highlighting libraries I have used.
Only two files and one function call and it just works, check it out!

```html
<link rel="stylesheet" href="/path/to/styles/default.css">
<script src="/path/to/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

The last and biggest change for my website is the addition of posts.
I have no plans for topics or how often I will publish posts yet, but now that a
place exists to put them, I might be more inclined to keep a record of interesting
things I come across during my daily frustrations with computers. :)

This post is mostly a record for myself, and an excuse to write something in order
to fill up space on the new site, but stay tuned for less boring posts later on.
