---
layout: page
title: About
description: 专注于Java底层及开源项目
keywords: Xiong Liu, 刘雄
comments: true
menu: 关于
permalink: /about/
---

仰慕「优雅编码的艺术」。

不断追求技术的深度和宽度，坚信能编码到70岁。

坚信熟能生巧，努力改变人生。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
