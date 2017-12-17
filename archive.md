---
layout: page
title: Archive
---

{% for post in site.posts %}
  {% assign currentDate = post.date | date: "%Y" %}
  {% if currentDate != myDate %}
  <h2>{{ currentDate }}</h2>
  {% assign myDate = currentDate %}
  {% endif %}

  * [ {{ post.title }} ]({{ post.url }}) &laquo; {{ post.date | date_to_string }}
{% endfor %}
