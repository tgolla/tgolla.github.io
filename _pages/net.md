---
layout: single
title: .NET
---

All things related to development in the .NET Framework.

{% assign posts = site.posts | where_exp: "post", "post.categories contains '.NET'" %}
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