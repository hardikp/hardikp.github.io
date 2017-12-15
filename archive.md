---
layout: page
title: Archive
permalink: /archive/
---

<section class="archive-post-list">

    {% for post in site.posts %}
        {% assign currentDate = post.date | date: "%Y" %}
        {% if currentDate != myDate %}
            {% unless forloop.first %}</ul>{% endunless %}
            <h1>{{ currentDate }}</h1>
            <ul>
            {% assign myDate = currentDate %}
        {% endif %}
        <li><a href="{{ post.url }}">{{ post.title }}</a> - <span style="color: gray;">{{ post.date | date: "%B %-d, %Y" }}</span></li>
        {% if forloop.last %}</ul>{% endif %}
    {% endfor %}

</section>
