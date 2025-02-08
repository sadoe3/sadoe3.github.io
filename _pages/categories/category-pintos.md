---
title: "PintOS"
layout: archive
permalink: categories/pintos
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.pintos %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
