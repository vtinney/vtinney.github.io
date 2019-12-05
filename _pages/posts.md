---
layout: archive
title: "Blog posts"
permalink: /posts/2019/
author_profile: true
---
{% include base_path %}

  <ul>{% for post in site.posts %}
    {% include archive-single-posts.html %}
  {% endfor %}</ul>
