---
layout: default
title: Blog
permalink: /blog/
---

# Blog

Welcome to the research and technical blog! For more about me and my work, see the [About](/about/) or [Research](/research/) pages.

{% for post in site.posts %}
---
### [{{ post.title }}]({{ post.url }})
<small>Posted {{ post.date | date: "%B %d, %Y" }}</small>
<br>
{{ post.excerpt | strip_html | truncate: 180 }}
<br>
{% endfor %}
