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
  justify-content: center;
  gap: 0.3rem;
  min-width: 115px;
  height: 24px;
  padding: 0 0.6rem;
  background: #f8f9fa;
  color: white;
  text-decoration: none;
  border-radius: 20px;
  border: 1px solid #dee2e6;
  transition: all 0.3s ease;
  font-weight: 500;
  font-size: 0.8em;
  box-sizing: border-box;
}

.taxonomy__item:hover {
  background: #e9ecef;
  color: white;
  text-decoration: none;
  transform: translateY(-2px);
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.taxonomy__name {
  font-weight: 500;
  letter-spacing: 0.3px;
  color: white;
  white-space: nowrap;
}

.taxonomy__count {
  background: #6c757d;
  color: white;
  padding: 0.08rem 0.35rem;
  border-radius: 8px;
  font-size: 0.75em;
  font-weight: 500;
  white-space: nowrap;
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
  color: white;
  border-color: #495057;
}

.dark-skin .taxonomy__item:hover {
  background: #495057;
  color: white;
}

.dark-skin .taxonomy__name {
  color: white;
}

.dark-skin .taxonomy__count {
  background: #6c757d;
  color: white;
}

/* Animation for smooth entrance */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.taxonomy__item {
  animation: fadeInUp 0.5s ease;
}

.taxonomy__item:nth-child(odd) {
  animation-delay: 0.1s;
}

.taxonomy__item:nth-child(even) {
  animation-delay: 0.2s;
}
</style>
