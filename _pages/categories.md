---
title: Categories
author: BabyK
date: 2022-03-15
category: Categories
layout: post
---

<section>
    {% for category in site.categories %}
        <a href="{{site.baseurl}}/pages/categories/{{category | first | downcase}}"> {{ category | first }}</a> <br>
    {% endfor %}
</section>



