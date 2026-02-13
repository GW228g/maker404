---
layout: default
title: Maker Blog
---
This blog documents how we apply the same maker and technology workflow in two environments: classroom teaching and real-world home projects. The goal is practical transfer. If a tool, process, or mindset works at home, we adapt it for students; if it works in school, we refine it for everyday problem-solving.

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
