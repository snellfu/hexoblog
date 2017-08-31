---
category: base
title: Java基础与干货---"".equal与equal("")
date: 2017-05-12 16:19:25
tags: Java
---

<!--more-->
### ```"".equal```与```equal("")```的区别？

   通常情况下两者相同，但是在与null对比时候不同,如下：


![](http://ocpue1vvp.bkt.clouddn.com/equalss.jpeg)

   所以推荐使用```"".equals()```可以避免空指针异常