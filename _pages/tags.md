---
layout: archive
title: "태그"
permalink: /tags/
author_profile: true
---

{% assign tags_max = 0 %}
{% for tag in site.tags %}
  {% if tag[1].size > tags_max %}
    {% assign tags_max = tag[1].size %}
  {% endif %}
{% endfor %}

<ul class="taxonomy__index">
  {% for i in (1..tags_max) reversed %}
    {% for tag in site.tags %}
      {% if tag[1].size == i %}
        <li>
          <a href="#{{ tag[0] | slugify }}">
            <strong>{{ tag[0] }}</strong> <span class="taxonomy__count">{{ i }}</span>
          </a>
        </li>
      {% endif %}
    {% endfor %}
  {% endfor %}
</ul>

{% for tag in site.tags %}
  <section id="{{ tag[0] | slugify | downcase }}" class="taxonomy__section">
    <h2 class="archive__subtitle">{{ tag[0] }}</h2>
    <div class="entries-list">
      {% for post in tag[1] %}
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
