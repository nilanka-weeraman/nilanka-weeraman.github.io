---
layout: default
title: Home
---

<ul class="post-list">
  {% for post in site.posts %}
    <li class="post-card">
      <a href="{{ post.url | relative_url }}">
        <h2>{{ post.title }}</h2>
        <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
      </a>
    </li>
  {% endfor %}
</ul>

