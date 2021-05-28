---
layout: page
title: About
description: 打码改变世界
keywords: Zheng xianglin, 郑祥林
comments: true
menu: 关于
permalink: /about/
---

我是郑祥林。

从小被人叫做祥林嫂，取名阿嫂一为警示，二为趣味。

学习总是痛苦的。在反复痛苦中磨练才是真理。

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
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
