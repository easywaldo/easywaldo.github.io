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

<div class="categories-cloud">
  <div class="category-badges">
    {% for i in (1..categories_max) reversed %}
      {% for category in site.categories %}
        {% if category[1].size == i %}
          <a href="#{{ category[0] | slugify }}" class="category-badge">
            <span class="category-name">{{ category[0] }}</span>
            <span class="category-count">{{ i }}</span>
          </a>
        {% endif %}
      {% endfor %}
    {% endfor %}
  </div>
</div>

{% for category in site.categories %}
  <section id="{{ category[0] | slugify | downcase }}" class="taxonomy__section">
    <h2 class="category-header">{{ category[0] }}</h2>
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

<style>
.categories-cloud {
  margin: 2rem 0;
}

.category-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin: 2rem 0;
}

.category-badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.3rem;
  min-width: 105px;
  height: 22px;
  padding: 0 0.5rem;
  background: #f8f9fa;
  color: #495057;
  text-decoration: none;
  border-radius: 20px;
  border: 1px solid #dee2e6;
  transition: all 0.3s ease;
  font-weight: 500;
  font-size: 0.75em;
  box-sizing: border-box;
}

.category-badge:hover {
  background: #e9ecef;
  color: #212529;
  text-decoration: none;
  transform: translateY(-2px);
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.category-name {
  font-weight: 500;
  letter-spacing: 0.3px;
  color: #495057;
  white-space: nowrap;
}

.category-count {
  background: #6c757d;
  color: white;
  padding: 0.05rem 0.3rem;
  border-radius: 8px;
  font-size: 0.7em;
  font-weight: 500;
  white-space: nowrap;
}

/* Category headers styling */
.category-header {
  color: white !important;
  font-size: 1.8em;
  font-weight: 600;
  margin: 3rem 0 1.5rem 0;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #dee2e6;
  position: relative;
}

.category-header::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 50px;
  height: 2px;
  background: #6c757d;
}

/* Dark theme compatibility */
.dark-skin .category-badge {
  background: #343a40;
  color: #f8f9fa;
  border-color: #495057;
}

.dark-skin .category-badge:hover {
  background: #495057;
  color: #ffffff;
}

.dark-skin .category-name {
  color: #f8f9fa;
}

.dark-skin .category-count {
  background: #6c757d;
  color: white;
}

.dark-skin .category-header {
  color: white !important;
  border-bottom-color: #495057;
}

.dark-skin .category-header::after {
  background: #6c757d;
}

/* Responsive design */
@media (max-width: 768px) {
  .category-badges {
    gap: 0.3rem;
  }
  
  .category-badge {
    padding: 0.3rem 0.6rem;
    font-size: 0.54em;
    transform: scale(0.55);
  }
  
  .category-header {
    font-size: 1.5em;
    margin: 2rem 0 1rem 0;
  }
  
  .taxonomy__section {
    transform: scale(0.65);
    padding: 0.5rem;
    margin-bottom: 1rem;
  }
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

.category-badge {
  animation: fadeInUp 0.5s ease;
}

.category-badge:nth-child(odd) {
  animation-delay: 0.1s;
}

.category-badge:nth-child(even) {
  animation-delay: 0.2s;
}

/* Hover effect for category sections */
.taxonomy__section {
  transition: all 0.3s ease;
  padding: 0.7rem;
  border-radius: 8px;
  margin-bottom: 1.4rem;
  transform: scale(0.7);
  transform-origin: left top;
}

/* Archive item styling for reduced size */
.archive__item {
  padding: 0.7rem 0;
  margin-bottom: 0.7rem;
}

.archive__item-title {
  font-size: 1.1em;
  margin-bottom: 0.35rem;
}

.archive__item-date {
  font-size: 0.8em;
}
</style>
