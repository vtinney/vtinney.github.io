---
layout: archive
title: "Blog posts"
permalink: /posts/
author_profile: true
---
{% include base_path %}


{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}
