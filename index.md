---
layout: page
title: kgeorge's programming blog
tagline: Computer Vision, Computer Graphics
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}"><h3>{{ post.title }}</h3></a>
    <blockquote>{{ post.excerpt }}<a href="{{ BASE_PATH }}{{ post.url }}">more...</a></blockquote>
    </li>
  {% endfor %}
</ul>



