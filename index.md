---
layout: page
title: Welcome
group: navigation
weight: 1
---

{% include JB/setup %}

> I just bought an e-copy of Learning Vaadin [...] and I'm happy somebody wrote this. [...] Why I think this book is great? I'm trying to introduce Vaadin to some of my coworkers, but Book of Vaadin is, as I'm told, too complex and too reference-style. This book is just what they need, something even a complete beginner could read and learn something. Here, in Slovenia, not many people know about Vaadin and a book like this will surely spread its use. I just want You guys to know your work is deeply appreciated in more places than you probably imagine :) Keep up the good work!

Mario Maric, Slovenia

During the writing of Learning Vaadin, I had many themes I wanted to write about: components data, SQL container filtering, component alignment and expand ration, separation of concerns between graphic designers and developers, only to name a few. Unfortunately, books are finite in space as well as in time and I was forced to leave out some interesting areas of Vaadin that couldn't fit in, much to my chagrin.

This site is meant to gap the bridge between what I wanted and what I could. Expect to see articles related to Vaadin!

## Sample Posts

This blog contains sample posts which help stage pages and blog data.
When you don't need the samples anymore just delete the `_posts/core-samples` folder.

    $ rm -rf _posts/core-samples

Here's a sample "posts list".

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


