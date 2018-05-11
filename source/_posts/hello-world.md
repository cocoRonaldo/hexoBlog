---
title: hexo 操作笔记
tags: 
    - hexo
    - note
comments: true
categories: 
    - Hexo
    - tech
---
# 参考资料
[官方文档](https://hexo.io/docs/)
[troubleshooting](https://hexo.io/docs/troubleshooting.html)
[ask](https://github.com/hexojs/hexo/issues)

# 目录
    [1.]基本用法
    [2.]插入html
    [3.]插入link和图片
    [4.]插入流程图
    [5.]插入公式
    [6.]引用
    [7.]data files

>1.1 标题 # ## ###
## 基本用法

>1.2 变粗 ** **

**变粗**

{% blockquote %}
>1.3 斜体 * * 
{% endblockquote %}
*斜体*

>1.4 中间划线 ~~ ~~

~~中间划线~~

>1.5 下划线不知道怎么搞
下划线 不知道怎么搞 ++我很好++
++22222++
==这个是区域==
==是多少==

>1.6 分割线 ---

---


>1.7 重点符号 -

- 重点符号

>1.8 加个框框 - [ ]

- [ ] 重点符号

>1.9 加个x框框 - [x]

- [x] 未完成符号

>1.10 排序


># 2.1 插入html

```
todo: 图片 图片并排等
```
<img src="" alt="" align="" border="" height="" width="" >
<a href="aaa"> 跳转</a>
<html>
<!--在这里插入内容-->
haodi
</html>

# 3 插入link 和 图片
[link](https://note.youdao.com/)

# 4 插入流程图

![image](https://note.youdao.com/favicon.ico)

# 5 插入公式

# 6 引用

{% youtube hY7M5jjJ9mM %}

{% blockquote %}
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Pellentesque hendrerit lacus ut purus iaculis feugiat. Sed nec tempor elit, quis aliquam neque. Curabitur sed diam eget dolor fermentum semper at eu lorem.
{% endblockquote %}

## quote from an article on the web
{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}

# 7 data files

<% for (var link in site.data.menu) { %>
    <a href="<%= site.data.menu[link]%>">   <%= link %> </a>
<% } %>


# 厉害 #
    我的过
