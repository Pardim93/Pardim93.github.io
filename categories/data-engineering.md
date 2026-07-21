---
layout: default
title: Data Engineering
---
<div class="category-page">
  <h1>Data Engineering</h1>
  <p>Artigos sobre engenharia de dados: Airflow, Kafka, dbt, Delta Lake e pipelines.</p>

  <div class="category-list">
    <ul>
      {% assign cat_posts = site.posts | where: "categories", "data-engineering" %}
      {% for post in cat_posts %}
      <li>
        <span class="cat-date">{{ post.date | date: "%d/%m/%Y" }}</span>
        <a class="cat-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </li>
      {% endfor %}
    </ul>
  </div>
</div>
