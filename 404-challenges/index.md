---
layout: default
title: 404 Challenges
---

<div class="cards">
  {% for challenge in site.challenges %}
    <div class="card">
      <h2>{{ challenge.title }}</h2>
      <p class="date">📅 {{ challenge.date | date: "%B %-d, %Y" }}</p>
      <p>{{ challenge.excerpt }}</p>
      <a href="{{ challenge.url }}" class="button">View Challenge »</a>
    </div>
  {% endfor %}
</div>
