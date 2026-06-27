---
layout: default
title: Blog
permalink: /blog/
---

# Blog

Welcome to my research and technical blog. This space highlights reflections on doctoral study, research documentation, embedded systems work, and project-based learning. You can also explore my [Research](/research/) page for current work and projects.

{% for post in site.posts %}
---
### [{{ post.title }}]({{ post.url }})
<small>Posted {{ post.date | date: "%B %d, %Y" }}</small>
<br>
{{ post.excerpt | strip_html | truncate: 180 }}
<br>
{% endfor %}
