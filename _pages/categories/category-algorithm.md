---
title: "Algorithm"
layout: archive
permalink: categories/algorithm
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories['Algorithm'] %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}