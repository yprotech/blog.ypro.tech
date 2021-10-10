---
layout: toppage
title: blog.ypro.tech
---

<p>
  {% for post in site.posts %}
    <div class="card mt-3">
      <div class="card-body">
        <h5 class="card-title">{{ post.title }}</h5>
      </div>
      <div class="card-footer text text-end fst-italic">
        <a href="{{ post.url }}">czytaj dalej ...</a>
      </div>
    </div>
  {% endfor %}
</p>
