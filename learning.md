---
layout: default
title: 学习资料
permalink: /learning/
---

# 学习资料

这里记录计算机视觉、机器人、模型部署和编程工具等学习内容。

{% assign sorted_learning = site.learning | sort: "date" | reverse %}

{% for note in sorted_learning %}

## [{{ note.title }}]({{ note.url | relative_url }})

- 分类：{{ note.category }}
- 更新时间：{{ note.date | date: "%Y-%m-%d" }}
- 标签：{{ note.tags | join: "、" }}

{{ note.description }}

{% endfor %}