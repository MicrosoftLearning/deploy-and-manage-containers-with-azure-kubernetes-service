---
title: Online Hosted Instructions
permalink: index.html
layout: home
---

# Content Directory

Hyperlinks to each of the lab exercises and demos are listed below.

## Labs

{% assign exercise = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Exercise |
| --- | 
{% for activity in exercise  %} | [{{ activity.exercise.title }}{% if activity.exercise.type %} - {{ activity.exercise.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

