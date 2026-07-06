---
layout: default
title: 学习资料
permalink: /learning/
---

# 学习资料

这里记录计算机视觉、机器人、模型部署和编程工具等学习内容。

{% assign sorted_learning = site.learning | sort: "date" | reverse %}

{% for note in sorted_learning %}
{% if note.title %}

## [{{ note.title }}]({{ note.url | relative_url }})

{% if note.category %}- 分类：{{ note.category }}{% endif %}
{% if note.date %}- 更新时间：{{ note.date | date: "%Y-%m-%d" }}{% endif %}
{% if note.tags %}- 标签：{{ note.tags | join: "、" }}{% endif %}

{{ note.description }}

{% endif %}
{% endfor %}
