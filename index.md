<header>
<i>>Welcome to mikhayeeer's tech blog</i>
</header>

# 文章列表

## 按标签分类
{% for tag in site.tags %}
### {{ tag[0] }}
<ul>
  {% for post in tag[1] %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}
    </li>
  {% endfor %}
</ul>
{% endfor %}

## 按发布日期分类
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}
    </li>
  {% endfor %}
</ul>

## 全部博客标题
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

# 欢迎来到我的博客

# 推荐阅读
> 一些个人很喜欢的参考资料以供查看

# 友情链接

CSDN: [blog.csdn\Topsort](https://blog.csdn.net/Topsort)

<footer>

</footer>
