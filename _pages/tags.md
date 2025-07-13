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

<div class="tags-cloud">
  <p class="page__lead">전체 {{ site.tags.size }}개의 태그가 있습니다.</p>
  
  <div class="taxonomy__index">
    {% for i in (1..tags_max) reversed %}
      {% for tag in site.tags %}
        {% if tag[1].size == i %}
          <a href="/tags/{{ tag[0] | slugify }}/" class="taxonomy__item">
            <span class="taxonomy__name">{{ tag[0] }}</span>
            <span class="taxonomy__count">{{ i }}</span>
          </a>
        {% endif %}
      {% endfor %}
    {% endfor %}
  </div>
</div>

<style>
.tags-cloud {
  margin: 2rem 0;
}

.taxonomy__index {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin: 2rem 0;
}

.taxonomy__item {
  display: inline-flex;
  align-items: center;
  gap: 0.3rem;
  padding: 0.4rem 0.8rem;
  background: #f8f9fa;
  border-radius: 20px;
  text-decoration: none;
  color: #495057;
  border: 1px solid #dee2e6;
  transition: all 0.3s ease;
  font-size: 0.9em;
}

.taxonomy__item:hover {
  background: #e9ecef;
  color: #212529;
  text-decoration: none;
  transform: translateY(-2px);
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.taxonomy__name {
  font-weight: 500;
}

.taxonomy__count {
  background: #6c757d;
  color: white;
  padding: 0.1rem 0.4rem;
  border-radius: 10px;
  font-size: 0.8em;
  font-weight: 500;
}

/* Dark theme styles */
@media (prefers-color-scheme: dark) {
  .taxonomy__item {
    background: #343a40;
    color: #f8f9fa;
    border-color: #495057;
  }
  
  .taxonomy__item:hover {
    background: #495057;
    color: #ffffff;
  }
  
  .taxonomy__count {
    background: #6c757d;
  }
}

/* Dark skin compatibility */
.dark-skin .taxonomy__item {
  background: #343a40;
  color: #f8f9fa;
  border-color: #495057;
}

.dark-skin .taxonomy__item:hover {
  background: #495057;
  color: #ffffff;
}

.dark-skin .taxonomy__count {
  background: #6c757d;
}
</style>
