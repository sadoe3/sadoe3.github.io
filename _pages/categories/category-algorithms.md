---
title: "Algorithms"
layout: archive
permalink: categories/algorithms
author_profile: true
---

{% assign posts = site.categories.algorithms %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
