---
title: 一个令人头秃的Bug
subtitle: 同样的代码跑在不同环境也会出错
author:
  nick: Daylight
  link: 'http://www.daylight.press/'
editor:
  name: Daylight
  link: 'http://www.daylight.press/'
date: 2021-01-19 19:33:42
tags: [iOS, PDFim, Bug, 头秃]
categories:
    - [Coding]
cover: /img/posts/a_very_wired_bug/cover.jpg
---
最近遇到了一个非常“神奇”的bug。

事情是这样的。我们在iOS上做了一个PDF Viewer，底层依赖于PDFium。在这之上，我们对PDFium的一些API进行了包装，并且定义了一系列UI。本质上，我们public了一个ViewController。Partner只要继承这个ViewController，并且调用相关的API就能将这个PDF Viewer嵌入到他们的APP中，集成完整的PDF阅读与编辑操作。

有一个partner，我们暂且把他称作U。U报了一个bug，声称他们在搜索PDF文件内容的时候，只会根据输入关键词的首字母来搜索，忽略了完整的单词。例如，输入"Hello"，高亮的是“H”的搜索结果。而这个问题，其余的partner，包括我们自己的test app，都没有遇到过。更令人窒息的是，U直接在项目中导入了我们编译好的.a文件。他们没办法debug我们的源码。

我们不能reproduce，他们不能debug。

气氛逐渐尴尬了起来。

一开始，我们认为问题出在搜索框那里。因为搜索的功能已经稳定运行了很久，也没有其他partner报过类似的问题；并且高亮的结果仍然是搜索首字母的结果，仍然是“正确的”；我们大胆地猜测，一定是U那边把输入的字符串截断了。鉴于不能reproduce和不能debug的尴尬气氛，我们能做的只有在代码里加了一些log，让U去试一试。

log的结果则打了我的脸。所有的log都显示传进来的字符串是完整的。

更深的层次，就是PDFium了，它对于我们来说是一个黑盒。如果PDFium出了问题，那就意味着所有的partner，包括我们的test app都会有类似的问题。然而我们从来没遇到过。于是，我们开始委婉地暗示。“其他的partner都没有遇到过。” “我们自己也测试过，没什么问题” “iOS的版本对不对” ~~“你们的t编译设备用的是交流电还是直流电”~~ ~~“会不会是天气不太好”~~

我们只能继续在更深层次的调用，也就是PDFium里，继续加log。

最后也确实找到了问题的原因。在PDFium的`ExtractSubString`函数里，对关键词长度的判断返回了1。但是，在`ExtractSubString` 内部，只调用了STL中的函数。所以我们只能大胆猜测，U那边的编译环境出了问题。而具体是什么问题，我们仍未可知。

等U找到了具体原因，我再来更新。

2021/01/20 update
原因找到了。U那边的APP，为了兼容一些东西，并没有使用iOS标准的STL，而是自己编译的一套。原本在iOS上，一个wchar是32bit长度的，PDFium会根据平台的不同主动对其进行适配。但是在U使用的STL中，一个wchar是16bit长度的。这就导致了，在PDFium中，对wchar指针所指向的字符串，对这个字符串的长度计算出错了。

传入PDFium的字符串是以UTF-16 Little End编码的，PDFium会将其转换为32bit长度，仍然以小端编码，即在末尾补零。但是在传入U那边的STL之后，会以16bit为一个字符。这就导致了，在调用wcslen计算字符串长度时，读取第17-32bit之后，wcslen会认为已经读到字符串结尾。因为一直都在输入英文做测试，所以才会出现永远搜索第一个字符的现象。

最后的解决方案是，在Xcode的build setting，other C++ flags子项中，加入了 -fshort-wchar。

看来以后测试的时候，要多尝试一些语言，包括emoji。