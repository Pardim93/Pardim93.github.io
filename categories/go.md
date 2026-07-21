---
layout: default
title: Go
---
<div class="category-page">
  <h1>Go</h1>
  <p>Artigos sobre Go (Golang): APIs, CLIs, scraping, generics e o ecossistema.</p>

  <div class="category-list">
    <ul>
      {% assign cat_posts = site.posts | where: "categories", "go" %}
      {% for post in cat_posts %}
      <li>
        <span class="cat-date">{{ post.date | date: "%d/%m/%Y" }}</span>
        <a class="cat-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </li>
      {% endfor %}
    </ul>
  </div>
</div>
