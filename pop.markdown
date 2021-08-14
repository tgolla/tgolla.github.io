---
layout: page
title: Pop
---

So here is one of those fun things.  When I was a kid growing up in Pennsylvania my Grandfather owned a beer and soda pop distributorship in Kittanning, PA.  In the basement of his house along with the old coal shoot he had a walk in refrigerator/freezer where his grandkids could always find a case or two of orange and cherry pop. 

Fast forward to today.  Oklahoma is known for having the longest stretch of old route 66.  In the town of Arcadia you will find the round barn ([http://www.arcadiaroundbarn.com](http://www.arcadiaroundbarn.com "Arcadia Round Barn"){:target="_blank"}) and POPS ([http://www.pops66.com](http://www.pops66.com "Pops"){:target="_blank"}), a modern day tourist attraction known for its stock of hundreds of different types of soda pop.  Anymore is become a detour for the family to stop and here is just some of what we have found.

{% assign posts = site.posts | where_exp: "post", "post.categories contains 'Pop'" %}
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
