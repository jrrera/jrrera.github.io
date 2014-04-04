---
layout: page
title: Thoughts on JavaScript, AngularJS, Python & App Engine.
tagline: Thoughts on JavaScript, AngularJS, Python & App Engine.
---
{% include JB/setup %}

<ul class="posts">
  <!--This code just lists out each posts title next a date with a link. Removing-->
<!--   {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %} -->

  <!--Added by Jon from: http://stackoverflow.com/questions/9794699/listing-all-the-blog-posts-with-content-with-jekyll-->
  {% for post in site.posts %}
    {% include JB/post_content %}
  {% endfor %}
</ul>