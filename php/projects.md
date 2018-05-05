---
title: PHP Projects
---

{% assign projects = site.projects | sort: 'title' %}
{% for project in projects %}
  <h2><a href="{{ project.url }}">{{ project.title }}</a></h2>
  <p>{{ project.description }}</p>
{% endfor %}
