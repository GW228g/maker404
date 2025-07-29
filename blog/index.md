---
layout: default
title: Blog
---

<h1>Maker Blog</h1>
<ul>
  {% for post in site.posts %}
    <li>
      <strong>{{ post.date | date: "%Y-%m-%d" }}</strong> –
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
