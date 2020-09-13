---
layout: archive 
permalink: /essays/
title: Essays
author_profile: false

description: "This little corner is a structured but miscellaneous collection of essays around technical topics"
toc: true
---
A little bit more of a description here. 

## Latest stories

<div class="grid__wrapper">
  {% assign collection = 'essays' %}
  {% assign posts = site[collection] | reverse %}
  {% for post in posts %}
    {% include archive-single.html type="grid" %}
  {% endfor %}
</div>
