---
layout: default
title: "Blog"
---

# Blog

[Home](./index.md) | [About](./about.md)

<ul>
{% for post in site.posts %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <small>{{ post.date | date: "%b %-d, %Y" }}</small>
  </li>
{% endfor %}
</ul>