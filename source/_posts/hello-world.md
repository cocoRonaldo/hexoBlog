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
* [1.基本用法](#1)
   * [1.1标题](#1.1)
   * [1.2变粗](#1.2)
   * [1.3斜体](#1.3)
   * [1.4中间划线](#1.4)
   * [1.5下划线](#1.5)
   * [1.6分割线](#1.6)
   * [1.7重点符号](#1.7)
   * [1.8加个框](#1.8)
   * [1.9加个x框](#1.9)
   * [1.10插入表格](#1.10)
* [2.插入html](#2)
    * [2.1 html插入图片](#2.1)
    * [2.2 html并排插入图片](#2.2)
* [3.插入link和图片](#3)
* [4.插入流程图](#4)
* [5.插入公式](#5)
* [6.引用](#6)
* [7.data files](#7)
* [8.插入表格](#8)

<h4 id="1.1"> 1.1 标题 # ## ###</h4>
### 基本用法
<h4 id="1.2"> 1.2 变粗 ** ** </h4>
**变粗**
<h4 id="1.3">1.3 斜体 * * </h4>
*斜体*

><h4 id="1.4">1.4 中间划线 ~~ ~~</h4> 

~~中间划线~~

<h4 id="1.5">1.5 下划线不知道怎么搞</h4>
下划线 不知道怎么搞 ++我很好++
++22222++
==这个是区域==
==是多少==

<h4 id="1.6">1.6分割线</h4>

---

<h4 id="1.7"> 1.7 重点符号 -</h4>

- 重点符号
- 一条一条的 
- 

<h4 id="1.8"> 1.8 加个框框</h4>

- [ ] 重点符号

<h4 id="1.9">1.9 加个x框框 - [x]</h4>

- [x] 未完成符号

<h4 id="1.10"> 1.10 插入表格</h4>

|时间|地点|人物|
|---|---|
|1|2|3|
|4|5|6|


 
<h3 id="2.1">2.1 插入html</h3>
<figure class="third">
    <img src="/images/cloudy.png">
    <img src="/images/cloudy.png">
</figure>
<img src="https://i.imgur.com/XAQCloy.jpg">

<div align="center">
    ![Alt text](/images/cloudy.png)
    ![Alt text](/images/cloudy.png)
</div>

<h3 id="2.2">2.2 并排插入</h3>
---
<div style="float:left;border:solid 1px 000;margin:2px;"><img src="/images/cloudy.png" width="200" height="260" ></div>
<div style="float:left;border:solid 1px 000;margin:2px;"><img src="/images/cloudy.png" width="200" height="60" ></div>
<br> </br>
<br> </br>
<br> </br>
---

<h4 id="3"> 3.插入link 和 图片</h4>
插入链接
[link](https://note.youdao.com/)
插入图片
![点击跳转](/images/cloudy.png "hexo 网站")

<a href="www.baidu.com">
    <img src="/images/cloudy.png">
</a>

### 直接跳转
<a href="/Main"> 你在哪里</a>

<h3 id="">4 插入流程图</h3>
```
npm install --save hexo-filter-flowchart
```
```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```

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

