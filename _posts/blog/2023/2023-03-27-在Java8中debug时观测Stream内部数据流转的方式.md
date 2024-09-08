---
title: '在Java8中debug时观测Stream内部数据流转的方式'
categories:
  - Blog
tags:
  - Java
---

你是否也曾觉得Java8好用，编写代码巨简洁、高效，但是调试就很痛苦，总是不能断点到你想要的位置，今天它来了（仅限idea编辑器，eclipse不适用）。

<!--more-->

大致**步骤**就是：
- 1、在Stream的第一行打断点；
- 2、点击跟踪当前流链按钮；

通过以上两步就可以图形化的看到Stream中的数据关系。

若是不明白怎么操作，请参考文中给出的地址，出自idea官网。
[https://www.jetbrains.com/help/idea/analyze-java-stream-operations.html](https://www.jetbrains.com/help/idea/analyze-java-stream-operations.html)