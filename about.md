---
title: about
permalink: /about/
---

<br>
<h1 class="owner-name">{{ site.owner.name}} </h1>
<h1 class="owner-name">{{ site.title }}</h1>

{{site.description}}

This blog is a collection of thoughts on a variety of topics. Low level aspects of computer science are going to be the main focus of my writing.

<div class="pagination">
  {% if site.owner.email %}
    <a href="mailto:{{ site.owner.email }}" class="social-media-icons"><i class="fa fa-2x fa-envelope" aria-hidden="true"></i></a>
  {% endif %}
  {% if site.owner.twitter %}
    <a href="{{ site.owner.twitter }}" class="social-media-icons"><i class="fa fa-2x fa-twitter" aria-hidden="true"></i></a>
  {% endif %}
  {% if site.owner.github %}
    <a href="{{ site.owner.github }}" class="social-media-icons"><i class="fa fa-2x fa-github" aria-hidden="true"></i></a>
  {% endif %}
  <a href="{{ site.url }}/assets/key.html" class="social-media-icons"><i class="fa fa-2x fa-key" aria-hidden="true"></i></a>
  <!-- <a href="{{ site.url }}/assets/key.html" class="social-media-icons"><i class="fas fa-key" aria-hidden="true"></i></a> -->
</div>
