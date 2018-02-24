---
title: hexo 问题记录
comments: true
toc: true
date: 2018-01-11 14:44:23
tags:
- hexo
categories:
- hexo
description: 记录使用hexo编辑博客时的问题
---
# markdown解析问题：
1. 标题的#号必须是连续的，比如大标题用#，下一级必须用##，不能使用###或其他。否则左侧目录顺序会乱。  
2. 列表后的文本排版会有问题。
    ```
    1. 123
    2. 123
    我是不属于列表中的文本。
    ```
    我们希望排版是上面这样，但是结果是
    1. 123
    2. 123
    我是不属于列表中文本（才怪）

  要修正错误，需要在最后一行前加2个空格，而不是一个tab或是4个空格。《==比如这行
3. 表格布局前要回车换一行
4. ```代码块后不能有换行

# gitment初始化评论报错Validate Error
问题：点击评论页的初始化按钮，报Validate Error错误。  
原因：github的issue label限制长度50字，而next在向github发送请求时传递的是本页面的url，长度会超过这个限制。  
解决：我使用的是next 5.x，定位代码位置  
`/theme/next/layout/_scripts/_third-party/comments/gitment.swig`:
![原先代码](http://ox34ivs2j.bkt.clouddn.com/%E5%8E%9F%E5%A7%8B%E6%83%85%E5%86%B5.png)修改26行的id，如下图：
![修改后代码](http://ox34ivs2j.bkt.clouddn.com/%E4%BB%A3%E7%A0%81%E4%BF%AE%E6%AD%A3.png)这样就解决了。