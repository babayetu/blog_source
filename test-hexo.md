---
title: test_hexo
date: 2016-03-04 16:53:15
tags: 测试
categories: 测试
---

{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}

{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}

{% codeblock %}
alert('Hello World!');
{% endcodeblock %}

{% include_code aaa.py lang:python aaa.py %}

{% link 搜狐新闻 http://news.sohu.com 我是提示 %}

![](/images/docker.jpeg)

{% for link in site.data.menu %}
  <a href="{{ link }}">{{ loop.key }}</a>
{% endfor %}
