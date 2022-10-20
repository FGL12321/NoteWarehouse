# Github如何稳定-高效-安全访问

[TOC]



## 1.0问题阐述

**为什么Github访问如此之慢？**

​		GitHub 的 CDN 域名遭到 DNS 污染，导致无法连接使用 GitHub 的加速分发服务器，才使得国内访问速度很慢。

**如何解决DNS污染？**

通过修改 Hosts 文件，将域名解析直接指向 IP 地址来绕过 DNS 的解析，以此解决污染问题。

**但是我要说的是，通过这样的方式并不能一劳永逸，同时也会产生下载缓慢或者突然无法访问的情况！**

## 2.0如何解决

今天给大家推荐一个超好用加速器，让你不使用魔法上网也可以实现高效稳定访问github

==Steam++是一款十分好用的软件，可以帮助我们几乎无损访问github！==

下载链接：[点击访问](https://gitee.com/fanggaolei/plug-in-warehouse/tree/master)

**点击Steam++_win_x86_v2.8.4.exe**

![image-20220827141358296](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220827141358296.png)

![image-20220827141412462](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220827141412462.png)

安装十分简单不做赘述！

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220827141638762.png)

![image-20220827141705756](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220827141705756.png)

**加速成功！**

## 3.0测试

**使用谷歌浏览器打开！**

![image-20220827142010347](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220827142010347.png)

文件push速度（这个视个人网速而定，如果关闭加速器依旧能有这个速度）：

![image-20220827144133099](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220827144133099.png)

**关键在于文件的下载和克隆速度：**

​		**这一点加速器并不能很好保证，但是可以通过插件实现高速下载。**

![image-20220827154607641](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220827154607641.png)

## 4.0克隆加速方法

**原链接**

```js
git clone https://github.com/tencentyun/qcloud-documents.git
```

**加速后的链接：将自己的链接进行修改即可**

```js
git clone https://github.com.cnpmjs.org/tencentyun/qcloud-documents.git
git clone https://hub.fastgit.org/tencentyun/qcloud-documents.git
git clone https://gitclone.com/github.com/tencentyun/qcloud-documents.git
```

## 5.0总结

​		**在使用和测试的过程中，可以说这款加速器可以保证我们正常的访问浏览以及下载一些相对较小的资源，例如项目源码等资料（这些文件本身就不大），因此是一款很好用且方便的软件。**