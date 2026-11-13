---
layout: page
title: /dev/mem
tagline: Things I'm working on/have worked on
---
{% include JB/setup %}

<ul class="posts">
  <li><span>2026</span> &raquo; Kubernetes with Talos on XCP-NG (WIP)</li>
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>



