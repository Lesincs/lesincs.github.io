---
title: hexo的优化和备份
date: 2020-01-24 20:27:12
tags: 
categories: 随想
thumbnail: /thumbnail/mike-scheid-639977-unsplash.jpg
---

### 前言

最近发现之前搭的`hexo`访问速度比较慢，因为`gitpage`本来速度就不是太稳定，而我设置的封面图片又比较大，所以导致页面加载起来卡卡的，遂决定优化一下，同时把`hexo`源码备份到了`hexo`分支，防止源码丢失又得重建。

<!-- more -->

### 图片压缩

我在`hexo`源码某个目录放了一堆`unsplash`图片用来作为封面，每次写博客随机取一张。下面是自己写的一个工具类，用于每次获取一个图片相对路径用于粘贴到博客正文里面。

```kotlin
object ThumbnailUtil {
    @JvmStatic
    fun main(arg: Array<String>) {
        val path = "/Users/lesincs/Hexo-Original/themes/icarus/source/thumbnail"
        val file = File(path)
        val prefix = "/thumbnail/"
        val names: MutableList<String> = ArrayList()

        file.listFiles()?.forEach {
            names.add(it.name)
        }

        val random = Random()
        println(prefix + names[random.nextInt(names.size)])
    }
}
```

但是我发现`thumbnail`目录下的图片都太大了，一般都是3m以上，甚至有的到达了十几m，实际上我们的`blog`只是为了美观，完全不需要这么大。于是决定将图片重新压缩下。

用`kt`写了一个简单压缩工具，用到了[thumbnailator](https://github.com/coobird/thumbnailator)这个第三方库，本来是想用`Luban`的，但是只能用于`Android Studio`,只能作罢。工具类代码:

````kotlin
object ImageCompressUtil {

    fun zip(fromDir: String, toDir: String) {

        val fileFromDir = File(fromDir)

        val srcPathsArray = Array<String>(fileFromDir.listFiles().size) {
            fileFromDir.listFiles()[it].absolutePath
        }

        Thumbnails
                .of(*srcPathsArray)
                .scale(0.25.toDouble())
                .outputQuality(0.2f)
                .toFiles(File(toDir), Rename.NO_CHANGE)

        print("压缩完成")
    }

}
````

这样压缩之后，发现图片只有几十或者几百k了，然后重新部署，访问了下，图片加载速度是快了不少。

### hexo源码备份

有时，想要在多设备上更新`blog`怎么办？这就需要将`hexo`源码放到某个位置，然后每次取到本地，更新源码了然后重新部署，同时只要保障服务器的源码是最新的就行了，这样就能达到多设备更新以及备份的作用。

步骤：

首先，在`hexo`文件夹下建立`git`仓库

`cd /hexo`

`git init`

建立`hexo` 分支并切换到`hexo`分支

`git chekout -b hexo`

提交文件

`git add .`

`git commit -m ‘add hexo source files’`

推送到远端

`git remote add origin  https://github.com/Lesincs/lesincs.github.io.git` 

`git push origin hexo` 

这样，`hexo`源文件就传到`github`上，需要的时候拉取下来就行了。

拉取的时候可以仅仅拉取hexo分支。

`git clone -b hexo https://github.com/Lesincs/lesincs.github.io.git`

### 最后

最后也不知道写啥了，测试一下`SM.MS`图床吧。

![](https://i.loli.net/2019/09/29/jLHfRFlv8hr9N5T.png)