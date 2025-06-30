---
layout: default.liquid
title: a blog
---
{% for post in collections.posts.pages %}
* [{{ post.title }}]({{ post.permalink }})
{% endfor %}
