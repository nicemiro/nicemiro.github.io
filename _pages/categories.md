---
title: Categories
author: BabyK
date: 2022-03-15
category: Categories
layout: post
---

<section>


    {% for category in site.categories %}
        <a href="{{site.baseurl}}/categories/{{category | first}}"> {{ category | first }}</a> <br>
    {% endfor %}

</section>



