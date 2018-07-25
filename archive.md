---
layout: page
title: Archive
permalink: /archive/
---

|:--------------|:---------------------|
| Date          | Blog Title           |
|:--------------|:---------------------|
{% for post in site.posts -%}
  | {{ post.date | date: "%Y-%m-%d" }} | [ {{ post.title }} ]({{ post.url }}) |
{% endfor %}
