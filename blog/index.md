---
layout: default
title: Maker Blog
---
<div class="blog-list">
  {% for post in site.posts %}
    <article class="blog-post-preview">
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p class="post-date">{{ post.date | date: "%B %-d, %Y" }}</p>
      <p>{{ post.excerpt | strip_html | truncatewords: 40 }}</p>
      <a href="{{ post.url }}" class="button">Read More »</a>
    </article>
  {% endfor %}
</div>
