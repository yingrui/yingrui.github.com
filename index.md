---
layout: page
title: 我正在思考!
tagline: 未来计算机是什么角色？
---
{% include JB/setup %}

整个博客是基于[Jekyll](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)和Github创建的，真的很棒

最近闲暇时，一直在做中文分词方面的工作。虽然中文分词作为中文处理中最基础的部分，目前已经非常完善，但是在应用中，一直没有发现符合软件工程思想的实现！那么我心目中的分词是什么样的呢？

## Post List
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


