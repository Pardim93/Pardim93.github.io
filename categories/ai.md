---
layout: default
title: Inteligência Artificial
---
<div class="category-page">
  <h1>Inteligência Artificial</h1>
  <p>Artigos sobre IA: MCP, prompt injection, vector databases e LLMs.</p>

  <div class="category-list">
    <ul>
      {% assign cat_posts = site.posts | where: "categories", "ai" %}
      {% for post in cat_posts %}
      <li>
        <span class="cat-date">{{ post.date | date: "%d/%m/%Y" }}</span>
        <a class="cat-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </li>
      {% endfor %}
    </ul>
  </div>
</div>
