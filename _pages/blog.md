---
layout: default
title: "Blog"
---

# Blog

<p>Welcome to my blog! Here are my latest posts:</p>

<ul>
{% for post in site.posts %}
  <li style="margin-bottom: 15px;">
    <a href="{{ post.url }}" style="font-weight: bold; font-size: 1.1em; color: #222;">{{ post.title }}</a><br>
    <small style="color: #666;">{{ post.date | date: "%b %-d, %Y" }}</small>
    <p>{{ post.excerpt | strip_html | truncate: 150 }}</p>
  </li>
{% endfor %}
</ul>
