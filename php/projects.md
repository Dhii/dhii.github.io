---
title: PHP Projects
---

{% assign projects = site.projects | sort: 'title' %}
{% for project in projects %}
  <h2>{{ project.title }}</h2>
  <p>{{ item.description }}</p>
  <p><a href="{{ item.url }}">{{ item.title }}</a></p>
{% endfor %}
