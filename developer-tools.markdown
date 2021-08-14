---
layout: page
title: Developer Tools
---

{% assign posts = site.posts | where_exp: "post", "post.categories contains 'Developer Tools'" %}
{% for post in posts %}
  <p>
    {{ post.date | date: "%Y-%m-%d" }}
    <h3>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h3>
  </p>
{% endfor %}
