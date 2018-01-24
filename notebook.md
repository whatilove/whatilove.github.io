---
layout: post
title: 地势坤，君子以厚德载物
subtitle: <span class="mega-octicon octicon-clippy"></span>&nbsp;&nbsp;笔记本
---

## Java IO、NIO、AIO
{% for post in site.categories.['io'] %}
- [{{ post.title }} ]({{ site.url }}{{ post.url }}) - {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
## 设计模式
{% for post in site.categories.['设计模式'] %}
- [{{ post.title }} ]({{ site.url }}{{ post.url }}) - {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
## Java Web
{% for post in site.categories.['web'] %}
- [{{ post.title }} ]({{ site.url }}{{ post.url }}) - {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
## JavaScript