---
title: gitalk抽风了
subtitle: 没有issue，不知所措
author:
  nick: Daylight
  link: 'http://www.daylight.press/'
editor:
  name: Daylight
  link: 'http://www.daylight.press/'
date: 2021-02-16 22:45:49
tags: [gitalk, 抽风]
categories:
    - [Coding]
---
# Gitalk再次抽风了，不知所措......
2021/2/17 update
原因终于找到了！
Gitalk用Github的OAuth Application来访问配置的repo并为每个post创建issue。我在Github的OAuth Application里，配置的call back URL是[http://www.daylight.press](http://www.daylight.press)，因此必须用完整的地址打开网页，才能让Gitalk初始化对应的issue，并且评论；省略www，用[http://daylight.press](http://daylight.press)访问就会出错。

