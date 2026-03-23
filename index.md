---
layout: default
title: Home
---

<ul class="post-list">
  {% for post in site.posts %}
    <li class="post-card">
      <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
      <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
      {% if post.tags %}
        <p class="post-tags">
          {% for tag in post.tags %}
            <a href="{{ '/tags/' | relative_url }}#{{ tag | slugify }}">#{{ tag }}</a>{% unless forloop.last %} {% endunless %}
          {% endfor %}
        </p>
      {% endif %}
    </li>
  {% endfor %}
</ul>

