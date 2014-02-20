---
date: 2014/2/20 16:54:39
layout: post
title: ExtJs4本地化的小bug
thread: 201402201654
categories: 前端
tags: ExtJs bug
---

使用`Ext.util.Format.number(value, '0.00')`时返回的数值，小数点是“,”，是ExtJs本地化的一个bug。

可在`ext-lang-zh_CN.js`中搜索decimalSeparator，值改为“.”，thousandSeparator值改为“,”