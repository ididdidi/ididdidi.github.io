---
title: Проекты
layout: default
category: main-menu
position: 1
lang: Ru
---

## Мои рабочие проекты
Проекты над которыми я активно работаю
{% assign work = site.projects | where: "tags", 'work' %}
{% include projects.html projects = work %}

## Мои студенческие работы
Это некоторые из моих первых работ. Они не несут какой либо особой ценности, но я сохранил их на память.
{% assign studies = site.projects | where: "tags", 'studies' %}
{% include projects.html projects = studies %}


