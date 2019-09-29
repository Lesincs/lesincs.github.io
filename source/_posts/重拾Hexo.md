---
title: 重拾Hexo
date: 2019-08-04 22:45:12
tags: 
categories: 随想
thumbnail: /thumbnail/jeff-brown-597952-unsplash.jpg
---

### 前言

闲来无事用GitPage部署了一个Hexo博客。之前其实也搞过，只不过是用的自己的腾讯云+自己买的域名。然后过期了之后就没搭理了。使用这一套的话自己不用去管服务器之类的，GitHub赛高。

<!-- more -->

### 过程

* 参考：[<使用 Hexo & GitPage 搭建博客>](<https://www.jianshu.com/p/08200e0ac81c>)
* 耗时：1.5 h左右
* 选用主题：[hexo-theme-icarus](<https://github.com/ppoffice/hexo-theme-icarus>)
* 遇到的坑：部署到GitHub时提示``Permisstion denied.``参考[git-ssh 配置和使用](https://segmentfault.com/a/1190000002645623)配置了SSH之后搞定。

### hexo特殊操作

* hexo多标签

  `tags: [标签1,标签2,标签3]`

* 自动摘要

  另起一行键入 `<!-- more -->`

* 插入封面

  在``title``处加上属性``thumbnail: /thumbnail/xxx.jpg``

* 本地部署

  `hexo s`

* 部署到``github``

  `hexo d`  

* 纯净部署

  `hexo clean & hexo d`

### 结语

有一段时间没写博客了，以后一定要多多输出。