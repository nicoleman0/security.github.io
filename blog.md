---
layout: default
title: Blog
permalink: /blog/
---

<div style="background-color: #333; padding: 10px; margin-bottom: 20px; text-align: center;">
  <a href="https://nicoleman0.github.io/security.github.io/" style="color: white; margin: 0 15px; text-decoration: none;">Home</a>
  <a href="https://nicoleman0.github.io/security.github.io/blog/" style="color: white; margin: 0 15px; text-decoration: none;">Blog</a>
  <a href="https://nicoleman0.github.io/security.github.io/#about-me" style="color: white; margin: 0 15px; text-decoration: none;">About</a>
</div>

# Blog
<ul class="post-list">
  {% for post in site.posts %}
    <li>
      <h2>
        <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
      </h2>
      <span class="post-meta">{{ post.date | date: "%B %-d, %Y" }}</span>
      <p>{{ post.excerpt }}</p>
    </li>
  {% endfor %}
</ul>
