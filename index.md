---
layout: toppage
title: blog.ypro.tech
---

<p>
  <div class="row">
    <div class="col-md-6">
      <div class="mt-3">
        <h4>Instrukcje</h4>
        {% for post in site.categories["Instrukcje"] %}
          <div class="card mt-3">
            <div class="card-body">
              <h5 class="card-title">{{ post.title }}</h5>
            </div>
            <div class="card-footer text text-end fst-italic">
              <a class="text-decoration-none" href="{{ post.url }}">czytaj dalej ...</a>
            </div>
          </div>
        {% endfor %}
      </div>
      <div class="mt-3">
        <h4>Drobne testy</h4>
        {% for post in site.categories["Drobne testy"] %}
          <div class="card mt-3">
            <div class="card-body">
              <h5 class="card-title">{{ post.title }}</h5>
            </div>
            <div class="card-footer text text-end fst-italic">
              <a class="text-decoration-none" href="{{ post.url }}">czytaj dalej ...</a>
            </div>
          </div>
        {% endfor %}
      </div>
    </div>
    <div class="col-md-6 mt-3">
      <h4>Pozostałe</h4>
      {% for post in site.categories["Pozostałe"] %}
        <div class="card mt-3">
          <div class="card-body">
            <h5 class="card-title">{{ post.title }}</h5>
          </div>
          <div class="card-footer text text-end fst-italic">
            <a class="text-decoration-none" href="{{ post.url }}">czytaj dalej ...</a>
          </div>
        </div>
      {% endfor %}
    </div>
  </div>
</p>
