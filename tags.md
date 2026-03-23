---
layout: default
title: Tags
permalink: /tags/
---

<h1>Tags</h1>

{% assign sorted_tags = site.tags | sort %}
<ul class="post-list">
  {% for tag in sorted_tags %}
    <li class="post-card">
      <a href="#{{ tag[0] | slugify }}">#{{ tag[0] }}</a> ({{ tag[1].size }})
    </li>
  {% endfor %}
</ul>

{% for tag in sorted_tags %}
  <h2 id="{{ tag[0] | slugify }}">#{{ tag[0] }}</h2>
  <ul class="post-list">
    {% for post in tag[1] %}
      <li class="post-card">
        <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
        <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
      </li>
    {% endfor %}
  </ul>
{% endfor %}
