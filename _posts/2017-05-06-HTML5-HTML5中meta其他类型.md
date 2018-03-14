---
layout:     post
title:      "HTML中meta其他类型"
subtitle:   "H5学习笔记"
date:       2017-05-06
author:     "xsnailcoder"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - HTML5
---

## meta其他类型
###meta标签分两大部分
* HTTP标题信息(HTTP-EQUIV)
* 页面描述信息(NAME)

### Keywords类型
作用：

* 设置搜索引擎当前网页的关键词, 在SEO（网络搜索优化）中非常重要, 能够提高搜索命中率

格式：

    <meta name="keywords" content="HTML5学习">
 
 
### Descriiption类型
作用：

* Description用来告诉搜索引擎你的网站主要内容

格式：

    <meta name="description" content="IT技术学习、iOS技术、HTML5技术">
    
### Author类型

作用：

* 标注网页的作者或制作组

格式：

    <meta name="author" content="xsnailcoder">
    
### Refresh类型
作用：

* 设置浏览器多久自动刷新一次页面并指向新页面

格式：

    <meta http-equiv="Refresh" content="3; URL=http://www.baidu.com/">
    
### Robots类型   
作用：

* Robots用来告诉搜索机器人哪些页面需要索引，哪些页面不需要索引。Content的参数有all、none、index、noindex、follow、nofollow。默认是all。

格式：

    <meta name="robots" value="index, follow"> 
    
属性：

* index
   
  告诉搜索引擎索引本页面，这是默认属性，如果不设置meta标签，搜索引擎默认会索引本页面。

* noindex

  告诉搜索引擎不要把本页展示在他们的搜索结果中。
  
* noimageindex

  禁止搜索引擎索引本页面上的图片，本页面上的图片不会显示在搜索结果中。

* none

  none是noindex,nofollow的缩写，告诉搜索引擎不要索引本页面，告诉爬虫不要索引本页面上的链接页面
  
* follow

  告诉搜索引擎可以从当前页面上找到链接，然后继续访问抓取下去
  
* nofollow

  告诉搜索引擎不允许从当前页面上找到链接, 拒绝其继续访问
  
* all

  告诉搜索引擎允许抓取当前页面, 并且允许从此页找到链接继续访问