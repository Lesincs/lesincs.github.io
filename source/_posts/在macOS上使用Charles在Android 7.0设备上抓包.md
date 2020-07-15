---
title: 在macOS上使用Charles在Android 7.0设备上抓包
date: 2020-07-15 10:57
tags: 
categories: Android
thumbnail: /thumbnail/marc-steenbeke-664660-unsplash.jpg

---

`Android 7.0`以上使用`Charles`对`Https`协议的抓包稍微繁琐了些，这里记录下步骤。

<!-- more -->

#### 1.下载Charles。

官网地址:[Charles官网](https://www.charlesproxy.com/)。`Charles`是收费软件，但有`30`天的免费体验时长。

#### 2.开启Https抓包

进入`Charles`，工具栏选中`Proxy`，点击`SSL Proxying Setting`。

然后增加一个如下图所示的配置，表示抓取所有域名的`443`端口的请求，`443`也就是`https`协议的默认端口。

![](https://i.loli.net/2020/07/15/LACzZneorlymdbv.png)

#### 3. 手机端WiFi开启代理

手机端需要和`mac`处在同一局域网，并且在`WiFi`处开启代理。代理服务器地址为`mac`的`ip地址`，可以通过如下方式查看：

1. 按住`Option` ，然后点击`mac`的`WiFi`图标，会出现如下界面，红框处就是`mac`的`ip地址`。

   ![](https://i.loli.net/2020/07/15/jDMRiXs1ZdrpoNY.png)

2. 第二种查看方式就是进入终端，输入 `ifconfig`命令，红框处就是`ip地址`。

   ![](https://i.loli.net/2020/07/15/abOKMz27t4ECuXi.png)

至于端口号，可以在`Charles`的`Proxy`->`Proxy Setting` 中看到，默认是`8888`。

代理设置完成之后，手机的网络请求边会通过`Charles`代理。此时，一般的`Http`请求都能抓取到了，但是`Https`还会存在问题，继续配置。

#### 4.手机端安装证书

1. 先将`Charles`的证书下载到`mac`本地。

2. 进入`Charles`，依次点击`Help`->`SSL Proxying`->`Save Charles Root Certificate `，然后选择存储路径之后，点击保存。随后就可以在保存位置看到一个名为`charles-ssl-proxying-certificate.pem`的文件。

3. 进入`Terminal`，`cd` 到刚才选择的存储目录。执行：

   ```bash
   openssl x509 -subject_hash_old -in charles-ssl-proxying-certificate.pem
   ```

   然后复制红框部分：

   ![](https://i.loli.net/2020/07/15/L3cfDRQ2pOTe5Fv.png)

4. 退出终端，将我们的`charles-ssl-proxying-certificate.pem`重新命名为

   `复制的内容.0`，拿我这个举例子也就是`fae0edc7.0`。

5. 将上面文件导入到`Android`手机的`/system/etc/security/cacerts` 目录。然后重启手机。大功告成。

### 5. 一些小坑

* `4.4`重命名这一步，我用`mac`自带的重名命方法，无法更改后缀。最终还是通过`Terminal`完成。执行的命令为：

  ```bash
  mv charles-ssl-proxying-certificate.pem fae0edc7.0
  ```

* `4.5`将证书导入到`Android`手机这一块，因为我的手机已经`root`过了，所以是将文件传到手机的`sdcard`目录之后，再在手机上通过`RE管理器`进行的导入。如果手机未`root`的话，应该可以通过`adb`进行一些目录授权再导入，因为时间有限，我也就没折腾了，具备一台已经`root`了的开发机还是很有必要的。

