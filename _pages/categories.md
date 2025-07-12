---
layout: archive
title: "카테고리"
permalink: /categories/
author_profile: true
---

{% assign categories_max = 0 %}
{% for category in site.categories %}
  {% if category[1].size > categories_max %}
    {% assign categories_max = category[1].size %}
  {% endif %}
{% endfor %}

<ul class="taxonomy__index">
  {% for i in (1..categories_max) reversed %}
    {% for category in site.categories %}
      {% if category[1].size == i %}
        <li>
          <a href="#{{ category[0] | slugify }}">
            <strong>{{ category[0] }}</strong> <span class="taxonomy__count">{{ i }}</span>
          </a>
        </li>
      {% endif %}
    {% endfor %}
  {% endfor %}
</ul>

{% for category in site.categories %}
  <section id="{{ category[0] | slugify | downcase }}" class="taxonomy__section">
    <h2 class="archive__subtitle">{{ category[0] }}</h2>
    <div class="entries-list">
      {% for post in category[1] %}
        <article class="archive__item">
          <h3 class="archive__item-title">
            <a href="{{ post.url | relative_url }}" rel="permalink">{{ post.title }}</a>
          </h3>
          <p class="archive__item-date">
            <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%Y년 %m월 %d일" }}</time>
          </p>
        </article>
      {% endfor %}
    </div>
    <a href="#page-title" class="back-to-top">{{ site.data.ui-text[site.locale].back_to_top | default: 'Back to Top' }} &uarr;</a>
  </section>
{% endfor %}
