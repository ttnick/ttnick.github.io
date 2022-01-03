---
title: Blog
layout: default
---
# Blog Overview 

There are {{ site.posts.size }} blog posts.

{{ site.posts | inspect }}

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
