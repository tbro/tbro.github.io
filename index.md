---
layout: default.liquid
title: a blog
---
{% for post in collections.posts.pages %}
* [{{ post.title }}]({{ post.permalink }})
{% endfor %}

<div id="footer" class="text-large">
Copyright © 2025 <a href="https://github.com/tbro">@tbro</a>.
</div>
