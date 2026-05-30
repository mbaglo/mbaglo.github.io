---
title: Blog
layout: default
---

## Blog

<ul>
{% for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <small>{{ post.date | date: "%B %d, %Y" }}</small>
    <p>{{ post.excerpt }}</p>
  </li>
{% endfor %}
</ul>
