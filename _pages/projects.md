---
layout: archive
title: "Projects"
permalink: /projects/
author_profile: true
---

{% include base_path %}

A selection of **recent and ongoing** projects. Click a card for more details.

<style>
.project-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1.75rem;
  margin-top: 2rem;
}
@media (max-width: 600px)  { .project-grid { grid-template-columns: 1fr; } }

.project-card {
  display: flex;
  flex-direction: column;
  text-decoration: none !important;
  color: inherit !important;
  border: 1px solid var(--global-border-color, #e6e6e6);
  border-radius: 8px;
  background: var(--global-bg-color, #fff);
  overflow: hidden;
  box-shadow: 0 1px 2px rgba(0,0,0,0.04);
  transition: transform .18s ease, box-shadow .18s ease, border-color .18s ease;
}
.project-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 18px rgba(0,0,0,0.08);
  border-color: var(--global-link-color, #52adc8);
}

.project-card__image {
  width: 100%;
  aspect-ratio: 5 / 3;
  object-fit: cover;
  display: block;
  background: #f3f4f6;
}

.project-card__body {
  padding: 1rem 1.1rem 1.15rem 1.1rem;
  display: flex;
  flex-direction: column;
  flex: 1;
}

.project-card__title {
  font-size: 1.05rem;
  font-weight: 700;
  line-height: 1.3;
  margin: 0 0 .35rem 0;
  color: var(--global-base-color, #2f7f93);
}

.project-card__status {
  display: inline-block;
  font-size: 0.72rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  padding: 0.15rem 0.5rem;
  border-radius: 999px;
  background: rgba(82, 173, 200, 0.12);
  color: var(--global-link-color, #2f7f93);
  margin-bottom: 0.55rem;
  align-self: flex-start;
}

.project-card__excerpt {
  font-size: 0.92rem;
  line-height: 1.5;
  color: var(--global-text-color, #444);
  margin: 0 0 .6rem 0;
  flex: 1;
}

.project-card__more {
  font-size: 0.88rem;
  font-weight: 600;
  color: var(--global-link-color, #52adc8);
  margin-top: auto;
}

.project-card__more::after { content: " \2192"; }
.project-card[data-external="true"] .project-card__more::after { content: " \2197"; }
</style>

{% assign sorted_projects = site.projects | sort: 'order' %}

<div class="project-grid">
{% for project in sorted_projects %}
  {% if project.external_url %}
    {% assign link_url = project.external_url %}
    {% assign external = "true" %}
    {% assign more_label = "Visit site" %}
  {% else %}
    {% assign link_url = project.url | relative_url %}
    {% assign external = "false" %}
    {% assign more_label = "More" %}
  {% endif %}

  <a class="project-card" data-external="{{ external }}" href="{{ link_url }}"{% if project.external_url %} target="_blank" rel="noopener"{% endif %}>
    {% if project.image %}
      <img class="project-card__image" src="{{ project.image | relative_url }}" alt="{{ project.title | escape }}" loading="lazy">
    {% endif %}
    <div class="project-card__body">
      {% if project.status %}<span class="project-card__status">{{ project.status }}</span>{% endif %}
      <h3 class="project-card__title">{{ project.title }}</h3>
      <p class="project-card__excerpt">{{ project.excerpt }}</p>
      <span class="project-card__more">{{ more_label }}</span>
    </div>
  </a>
{% endfor %}
</div>
