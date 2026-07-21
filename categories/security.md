---
layout: default
title: Segurança
---
<div class="category-page">
  <h1>Segurança</h1>
  <p>Artigos sobre cybersegurança: ataques, defesas, OWASP, supply chain e autenticação.</p>

  <div class="category-list">
    <ul>
      {% assign cat_posts = site.posts | where: "categories", "security" %}
      {% for post in cat_posts %}
      <li>
        <span class="cat-date">{{ post.date | date: "%d/%m/%Y" }}</span>
        <a class="cat-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </li>
      {% endfor %}
    </ul>
  </div>
</div>
