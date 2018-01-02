---
layout: default
title: Blog
permalink: /blog/
---

# blog
_I don't know how to blog, but this is it._

<div class="posts">
      {% for post in site.posts %}{% if post.hidden != true %}<a href="{{ post.url }}" class="post-link">{{ post.title }}</a>
      {% endif %}{% endfor %}
</div>
