---
layout: page
title: Wiki
description: 吾生也有涯，而知也无涯。
keywords: 维基, Wiki
comments: false
menu: 维基
permalink: /wiki/
---

> 请直接查看<https://github.com/pezy/blog/wiki>

<ul class="listing">
{% for wiki in site.wiki %}
{% if wiki.title != "Wiki Template" %}
<li class="listing-item"><a href="{{ wiki.url }}">{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
</ul>
