---
title: ios
author: BabyK
date: 2022-03-15
category: Categories
layout: post
---

{% assign title = page.title | escape %}
{% assign collection = site.categories[title] | reverse %}


<section>
{% for index in collection %}
        <a href="{{site.baseurl}}{{index.url}}" name="{{ index.title}}">{{index.date | date : "%Y/%m/%d"}} - {{ index.title}}
        <span style="font-size:small" >( {{ index.titleEn }} )</span></a> <br>
{% endfor %}
</section>





