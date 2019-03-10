---
title: 百度OCR图片文字识别
date: 2018-3-23 22:37:00
comments: true
categories: [SpringBoot]
tags: [SpringBoot,ZUI,OCR]
---

前段时间对面项目组的实习生问如何把截图中的代码直接输到 Eclipse 中，因为他的项目组长给了他一份文档，而其中的示例代码全都是以截图形式保存在 Word 中的，让我想到了[百度云](https://cloud.baidu.com/)提供的各种人工智能应用接口中就包括了免费的图片文字识别接口调用，所以就简单写了个 WEB 程序，能够上传包含文字的图片并返回识别结果

<!-- more -->

代码比较简单，[已上传到 GitHub](https://github.com/1976841230/TOOLS)
项目演示地址：tools.kuranado.com/upload

用到的主要技术：
- 后端：Spring Boot
- 前端：[ZUI](http://zui.sexy/)
- 百度云图片文字识别接口：[https://ai.baidu.com/docs#/OCR-Java-SDK/top](https://ai.baidu.com/docs#/OCR-Java-SDK/top)
- 项目构建工具：Maven

下面简单演示项目：

访问 tools.kuranado.com/upload，界面如下：

![mark](http://imgblog.kuranado.com/blog/180323/5310KGF1cK.png)

从本地拖拽一张图片到上传界面，然后点击“开始上传”按钮即可开始上传图片，图片上传成功后，会返回文字识别结果，我选了张测试图片如下：

![mark](http://imgblog.kuranado.com/blog/180323/C4JEGI7LdH.png)

上传这张图片后识别结果如下：

![mark](http://imgblog.kuranado.com/blog/180323/mHefFjIDeG.png)

使用文本比较工具比较识别结果的准确性（左侧为原文本，右侧为图片中文本识别结果）：

![mark](http://imgblog.kuranado.com/blog/180323/Ame09b5mkL.png)

可以看到识别结果除了一些标点符号和大小写，其他算的上是相当精确的了