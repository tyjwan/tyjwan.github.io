---
layout: page
title: About
description: 一位愿意相信努力可以改变命运的程序员
keywords: tangyejun, 唐烨君
comments: true
menu: 关于
permalink: /about/
---

我是唐烨君，热爱技术，热爱coding。

座右铭: 靡不有初,鲜克有终。


## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'tyjwan' %}
<li>
哔哩哔哩up主：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ assets_base_url }}/assets/images/qrcode.jpg" alt="爱学习的程序员唐唐" />
</li>
{% endif %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
