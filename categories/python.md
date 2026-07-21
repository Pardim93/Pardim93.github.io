---
layout: default
title: Python
---
<div class="category-page">
  <h1>Python</h1>
  <p>Todos os artigos sobre Python: versões, features, ferramentas, automação e boas práticas.</p>

  <div class="category-list">
    <ul>
      {% assign cat_posts = site.posts | where: "categories", "python" %}
      {% for post in cat_posts %}
      <li>
        <span class="cat-date">{{ post.date | date: "%d/%m/%Y" }}</span>
        <a class="cat-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </li>
      {% endfor %}
    </ul>
  </div>
</div>
