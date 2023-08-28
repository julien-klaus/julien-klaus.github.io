---
permalink: /
title: ""
excerpt: "About me"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

# About me

I'm a theoretical computer scientist, compiler builder and great organizer who loves to transform things in Python, think about algorithms, organize events or just travel and discover new things.


# Latest blog posts

{% include base_path %}
{% assign posts_to_display = 3 %}
{% assign post_count = 0 %}
{% for post in site.posts %}
  {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
  {% if year != written_year %}
    {% capture written_year %}{{ year }}{% endcapture %}
  {% endif %}
  {% if post_count < posts_to_display %}
    {% include archive-single.html %}
    {% assign post_count = post_count | plus: 1 %}
  {% endif %}
{% endfor %}