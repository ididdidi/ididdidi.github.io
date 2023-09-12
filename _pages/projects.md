---
layout: default
category: main-menu
position: 1
lang: En
---

## My work projects
Projects I'm working on
{% assign work = site.projects | where: "tags", 'work' %}
{% include projects.html projects = work %}

## My studies projects
These are some of my first works. They don't have any special value, but I kept them as a keepsake.
{% assign studies = site.projects | where: "tags", 'studies' %}
{% include projects.html projects = studies %}

