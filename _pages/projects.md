---
layout: page
title: Data Science Projects
permalink: /projects/
description: 
nav: true
nav_order: 1
display_categories: []  # Remove specific categories
horizontal: false
---
<!-- pages/projects.md -->
<div class="projects">
{%- assign sorted_projects = site.projects | sort: "importance" -%}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="col-sm">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include projects.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
</div>
