---
layout: page
title: 我正在思考!
tagline: 未来计算机是什么角色？
---
{% include JB/setup %}

这个博客是基于[Jekyll](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)和Github创建的，真的很棒

我还完全是Jekyll新手，下一步准备找到上传图片的方法，哈哈

## Post List
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


