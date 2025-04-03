---
title: "PintOS"
layout: archive
permalink: categories/project1
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.project1 %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
