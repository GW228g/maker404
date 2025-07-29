---
layout: default
title: Maker Blog
---

<div class="cards">
  {% for post in site.posts %}
    <div class="card">
      <h2>{{ post.title }}</h2>
      <p class="date">📅 {{ post.date | date: "%B %-d, %Y" }}</p>
      <p>{{ post.excerpt }}</p>
      <a href="{{ post.url }}" class="button">Read More »</a>
    </div>
  {% endfor %}
</div>
