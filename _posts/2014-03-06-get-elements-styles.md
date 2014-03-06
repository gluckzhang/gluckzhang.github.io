---
date: 2014/3/6 11:24:29 
layout: post
title: JavaScript获取DOM样式方法
thread: 201403061124
categories: 前端
tags: JavaScript
---

首先要看一下样式定义的3中方式：

- 外部样式(External Style Sheet)

> 以css为扩展名的文件，作用范围可以是多张网页，或整个网站，甚至不同的网站。与网页链接后，才能应用。

```
<link rel="stylesheet" href="xxx.css" type="text/css" />
```

- 嵌入式样式(Internal Style Sheet)

> 包含在网页内部的样式设置，它的作用范围仅限于嵌入的网页。

```
<style>
	.some-class {
		...
	}
</style>
```

- 内联式样式(Inline Style)

> 在HTML文档中，内联式样式表的格式化信息直接插入所应用的网页元素的HTML标签中，作为其HTML标签的属性参数。严格地说，内联样式表称不上表，仅仅是一条HTML标记。

```
<div style="width: 400px;"></div>
```

当同时定义某一种样式时，优先级是内联 `>` 嵌入式样式 `>` 外部样式。

###DOMObject.style.xxx
这种方法最容易想到，例如获取一个iframe的宽度，我们可以这样写：

	var width = document.getElementById("contentFrame").style.width;

但`DOMObject.style.xxx`这种方法**只能**够获取到**内联样式**，如果width不是直接在iframe标签里面定义的，则会取到空值。

###document.defaultView.getComputedStyle()
获取某元素最终呈现出来的样式，可以通过`getComputedStyle()`方法实现，还按上面的需求来写：

	var contentFrame = document.getElementById("contentFrame");
	var width = document.defaultView.getComputedStyle(contentFrame).width;

`getComputedStyle()`方法有两个参数，第一个参数必选，是一个DOM对象；第二个参数可选，是需要获取的样式伪类（例如hover等）。
