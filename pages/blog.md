---
layout: default
title: "Blog"
permalink: /blog/
---

<h1>Blog</h1>

{% for post in paginator.posts %}
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <p>{{ post.excerpt }}</p>
{% endfor %}

{% if paginator.total_pages > 1 %}
  <nav>
    {% if paginator.previous_page %}
      <a href="{{ paginator.previous_page_path }}">Previous</a>
    {% endif %}
    {% if paginator.next_page %}
      <a href="{{ paginator.next_page_path }}">Next</a>
    {% endif %}
  </nav>
{% endif %}