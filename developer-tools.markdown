---
layout: page
title: Developer Tools
---

Through the years there have been several good list of tools available to make a developers life just a bit easier. These list seem to come and go with little review and/or substance on what the tool can do for you. I’ve put this list together in the form of blog entries to provide both more information and a means to search the list with tags. You can also expect the publish date to reflect the last time I took a close look at the tool. Enjoy and please notify me should you find a better tool or something out of date.

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
