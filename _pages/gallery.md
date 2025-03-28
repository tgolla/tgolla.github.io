---
layout: single
title: Gallery
permalink: /gallery/
description: Gallery of website header images.
---

<div class="header-gallery">
  {% for page in site.pages %}
    {% if page.header.image and page.title != "Gallery" %}     
      {% capture display_title %}
        {% if page.permalink == "/" %}
          Home
        {% else %}
          {{ page.title }}
        {% endif %}
      {% endcapture %}
      <div class="header-item">
        <a href="{{ page.url }}">
          <img src="{{ page.header.image }}" alt="{{ display_title }}">
        </a>
        <p>{{ display_title }}</p>
      </div>
    {% endif %}
  {% endfor %}
  {% for post in site.posts %}
    <div class="header-item">
      <a href="{{ post.url }}">
        <img src="{{ post.header.image }}" alt="{{ post.title }}">
      </a>
      <p>{{ post.title }}</p>
    </div>
  {% endfor %}
</div>
