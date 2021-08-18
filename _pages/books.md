---
layout: single
title: Books
---

Books, books and more books. I never seem to have a enough time to read half the books that come to my attention. What you will find here is a list of mostly technical, business and career development and self-help books most of which I have found to be useful in my life.

{% assign posts = site.posts | where_exp: "post", "post.categories contains 'Books'" %}
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
