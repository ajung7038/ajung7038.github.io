---
title: "Article"
layout: archive
permalink: categories/article
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories['Article'] %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}