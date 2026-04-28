---
title: Blog
layout: default
---
# Blog Overview 

<ul>
  {% for post in site.posts %}
    <li>
      <i>{{ post.date | date: "%B %-d, %Y" }}:</i> <a href="{{ post.url }}">{{ post.title }}{% if post.tagline %} ({{ post.tagline }}){% endif %}</a>
    </li>
  {% endfor %}
</ul>
