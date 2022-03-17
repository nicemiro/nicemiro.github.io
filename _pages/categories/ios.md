---
title: iOS
author: BabyK
date: 2022-03-15
category: Categories
layout: post
---

{% assign title = page.title | escape %}
{% assign collection = site.categories[title] | reverse %}

<section>

{{title}} <br>
========== <br>
{% for category in site.categories %}
<a> {{site.baseurl}}/pages/categories/{{category | first | downcase}} </a> <br>
<a>{{ category | first }} </a> <br>
{% endfor %}
</section>
