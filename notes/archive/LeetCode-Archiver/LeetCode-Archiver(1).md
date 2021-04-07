---
title: "LeetCode Archiver(1)：Scrapy框架和Requests库"
date: 2018-12-04T11:25:15+11:00
draft: false
categories: ["Python"]
---

## 简介

Scrapy官方文档对Scrapy的介绍如下：

Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。<br>其最初是为了页面抓取（更确切来说, 网络抓取）所设计的，也可以应用在获取API所返回的数据（例如 Amazon Associates Web Services ）或者通用的网络爬虫。

简而言之，Scrapy是基于Twisted库开发的，封装了http请求、代理信息、数据存储等功能的Python爬虫框架。

## 组件和数据流

下图是Scrapy官方文档中的架构概览图：

![Architecture](https://scrapy-chs.readthedocs.io/zh_CN/0.24/_images/scrapy_architecture.png)

图中绿色箭头表示<a href="#head">数据流</a>，其他均为组件。

### Scrapy Engine（引擎）
引擎负责控制数据流在系统的组件中流动，并在相应动作发生时触发事件。

### Scheduler（调度器）
调度器从引擎接收request并将其保存，以便在引擎请求时提供给引擎。

### Downloader（下载器）
下载器负责下载页面数据，并将其提供给引擎，而后再由引擎提供给爬虫。

### Spiders（爬虫）
Spider是由用户编写的用于**分析response**并**提取item**或额外**跟进url**的类。一个Scrapy项目中可以有很多Spider，他们分别被用于爬取不同的页面和网站。

### Item Pipeline（管道）
Item Pipeline负责处理被爬虫**提取出来的item**。可以对其进行数据清洗，验证和持久化（例如存储到数据库中）。

### Downloader middlewares（下载器中间件）
下载器中间件是在引擎及下载器之间的组件，用于处理下载器传递给引擎的response。更多内容请参考[下载器中间件](https://scrapy-chs.readthedocs.io/zh_CN/0.24/topics/downloader-middleware.html#topics-downloader-middleware)。

### Spider middlewares（爬虫中间件）
Spider中间件是在引擎及Spider之间的组件，用于处理爬虫的输入（response）和输出（items和requests）。更多内容请参考[爬虫中间件](https://scrapy-chs.readthedocs.io/zh_CN/0.24/topics/spider-middleware.html#topics-spider-middleware)。

### <a id="head"/> Data flow（数据流）</a>
Scrapy中的数据流由引擎控制，其过程如下:<br>
1.引擎打开一个网站，找到处理该网站的爬虫并向该爬虫请求要爬取的url。<br>
2.引擎从爬虫中获取到要爬取的url并将其作为request发送给调度器。<br>
3.引擎向调度器请求下一个要爬取的url。<br>
4.调度器返回下一个要爬取的url给引擎，引擎将url通过下载器中间件发送给下载器。<br>
5.下载器下载页面成功后，生成一个该页面的response对象，并将其通过下载器中间件发送给引擎。<br>
6.引擎接收从下载器中间件发送过来的response，并将其通过爬虫中间件发送给爬虫处理。<br>
7.爬虫处理response，并将爬取到的item及跟进的新的request发送给引擎。<br>
8.引擎将爬虫返回的item发送给管道，将爬虫返回的新的request发送给调度器。<br>
9.管道对item进行相应的处理。<br>
10.重复第二步，直到调度器中没有更多的request，此时引擎关闭该网站。<br>

## 安装

1.下载安装最新版的[Python3](https://www.python.org/downloads/)

2.使用pip指令安装Scrapy
```
pip3 install scrapy
```

## 创建项目

首先进入你的代码存储目录，在命令行中输入以下命令：
```
scrapy startproject LeetCode_Crawler
```
注意项目名称是不能包含连字符 '-' 的

新建成功后，可以看到在当前目录下新建了一个名为LeetCode_Crawler的Scrapy项目，进入该目录，其项目结构如下：
```
scrapy.cfg              #该项目的配置文件
scrapy_project          #该项目的Python模块
    __init__.py
    items.py            #可自定义的item类文件
    middlewares.py      #中间件文件
    pipelines.py        #管道文件
    settings.py         #设置文件
    __pycache__
    spiders             #爬虫文件夹，所有爬虫文件都应在该文件夹下
        __init__.py
        __pycache__
```

至此Scrapy项目的创建就完成了。


## 参考资料
<a href="https://scrapy-chs.readthedocs.io/zh_CN/0.24/" target="_blank">Scrapy官方文档</a>