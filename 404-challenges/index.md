---
layout: default
title: 404 Challenges
---

<h1>404 Challenges</h1>
<ul>
  {% for challenge in site.challenges %}
    <li>
      <strong>{{ challenge.date | date: "%Y-%m-%d" }}</strong> –
      <a href="{{ challenge.url }}">{{ challenge.title }}</a>
    </li>
  {% endfor %}
</ul>
