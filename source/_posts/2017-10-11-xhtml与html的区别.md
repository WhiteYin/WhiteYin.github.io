---
title: xhtml与html的区别
comments: true
toc: true
date: 2017-10-11 12:27:18
tags: 
- xhtml 
- html
description: xhtml与html的区别以及如何将html转换成xhtml
---
### 文档结构
* <!DOCTYPE>是强制的
* <html>中的XML namespace属性是强制的
* <html>、<head>、<body>也是强制的
### 元素语法
* 元素必须正确嵌套
* 元素必须始终关闭
* 元素必须小写
* 文档必须有一个根元素
### 属性语法
* 属性必须使用小写
* 属性值必须用引号包围
* 属性最小化也是禁止的
* 不允许属性简写
### 将html转换成xhtml
1. 添加一个xhtml <!DOCTYPE>到网页中
2. 添加xmlns属性到每个页面的html元素中
3. 改变所有元素为小写
4. 关闭所有空元素
5. 修改所有属性名为小写
6. 所有属性值添加引号
