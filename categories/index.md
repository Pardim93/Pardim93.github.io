---
layout: default
---
<div class="category-page">
  <h1>{{ page.title }}</h1>
  {{ content }}

  {% assign categories = "python,go,security,mobile,data-engineering,ai,docker,php,productivity" | split: "," %}

  {% for cat in categories %}
    {% assign cat_posts = site.posts | where: "categories", cat %}
    {% if cat_posts.size > 0 %}
    <div class="category-list">
      <h2 id="{{ cat }}"><a href="#{{ cat }}">{{ cat | capitalize }}</a></h2>
      <ul>
        {% for post in cat_posts %}
        <li>
          <span class="cat-date">{{ post.date | date: "%d/%m/%Y" }}</span>
          <a class="cat-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        </li>
        {% endfor %}
      </ul>
    </div>
    {% endif %}
  {% endfor %}
</div>
