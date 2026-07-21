---
layout: default
title: Mobile
---
<div class="category-page">
  <h1>Mobile</h1>
  <p>Artigos sobre desenvolvimento mobile: React Native, Flutter, Jetpack Compose e Kotlin Multiplatform.</p>

  <div class="category-list">
    <ul>
      {% assign cat_posts = site.posts | where: "categories", "mobile" %}
      {% for post in cat_posts %}
      <li>
        <span class="cat-date">{{ post.date | date: "%d/%m/%Y" }}</span>
        <a class="cat-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </li>
      {% endfor %}
    </ul>
  </div>
</div>
