---
title: 解决Gradle下载自身缓慢的问题
date: 2020-10-13 23:38
tags: 
categories: Android
thumbnail: /thumbnail/joseph-barrientos-20264-unsplash.jpg
---

本文讨论的是 Gradle 下载自身缓慢的问题，也就是在 Build 之前，由于不同 Android 项目依赖不同的 Gradle 版本，如果本地 Wrapper 文件夹中不存在该版本，则需要联网下载，而这个下载，是极其缓慢的，并且代理软件对其的下载速度是无影响的。这里有两种方式去解决。

<!-- more -->

### 1. 手动下载大法

这个方法应该搞 Android 开发的人都尝试过。也就是新启动一个项目的时候，在 Android Studio 下载 Gradle 阶段，我们强制杀掉 Android Studio ，并从项目的 `gradle-wrapper.properties` 文件中获取到对应的 Gradle 版本；然后 `Google`搜索`gradle distributions` ,进入 https://services.gradle.org/distributions/ 页面进行对应 Gradle 的下载。下载完成之后，将其移动到对应的 Wrapper 目录，再重新打开项目即可。

### 2. 通过命令行代理

这个方法是我在知乎的一个答案发现的，[原答案链接](大家都是怎样处理Gradle中的这个文件下载慢的问题的？ - 啊鱼的回答 - 知乎 https://www.zhihu.com/question/37810416/answer/156162582) 。他的原理是如果使用 Android Studio 来下载 Gradle ，由于权限问题，即使你的代理软件设置全局模式，下载 Gradle 也不会走代理，但是用命令的方式是可以做到的。

具体的代码如下：

```
./gradlew -DsocksProxyHost=127.0.0.1 -DsocksProxyPort=7890 tasks
```

其中的 port 根据自己的代理软件进行替换，我用软件的是 ClashX，默认代理端口是 7890，经过测试，使用该方式之后，之前一直下载缓慢的 Gradle 也能在终端快速的下载。

### 吐槽

不得不说，作为一名中国的开发者，因为墙的问题，在 Android 开发的道路上得多走好多的弯路。希望有一天，这个世界会没有墙的存在。