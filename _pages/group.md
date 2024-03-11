---
layout: default
permalink: /group/
title: group
description:
nav: true
nav_order: 4
display_categories: [Postdocs and PhD Students, MS and BS Students, Former Students]
horizontal: false
---

# **R**esponsible **AI** for **S**cience and **E**ngineering (RAISE) Group

I lead the RAISE group where I am fortunate to work with an amazing set of students and collaborators.
We work on foundational topics relating to machine learning and optimization, privacy and fairness.
We often ground our research in applications at the intersection of physical sciences and energy, as
well as policy and decision making.



<!-- pages/projects.md -->
<div class="projects">
{%- if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized projects -->
  {%- for category in page.display_categories %}
  <h2 class="category">{{ category }}</h2>
  {%- assign categorized_projects = site.group | where: "category", category -%}
  {%- assign sorted_projects = categorized_projects | sort: "importance" %}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include group.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
  {% endfor %}

{%- else -%}
<!-- Display projects without categories -->
  {%- assign sorted_projects = site.group | sort: "importance" -%}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include group.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
{%- endif -%}
</div>