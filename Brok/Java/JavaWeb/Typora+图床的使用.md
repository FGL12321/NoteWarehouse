[TOC]



# 搭建Typora+PicGo+腾讯云COS笔记保姆级教程

## 1.PicGo的下载

​		  网上有很多下载的方式，有的是下载速度慢，有的是无法获取连接，这里给大家分享一个优质的下载链接：

​          **山东大学镜像下载：**https://mirrors.sdu.edu.cn/github-release/Molunerfinn_PicGo/v2.3.0/

​		 **windos用户选择：**

![image-20220820111946454](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820111946454.png)

​		安装完成后，我们可以选择gitee，github提供的图床服务。

​		由于gitee需要实名认证，github访问速度较慢，所以这里推荐使用腾讯云COS作为存储桶

## 2.Node.js的下载

​		Node.js的下载，是**针对需要下载gitee插件的用户设置的**。如果我们选择gitee作为存储桶则需要下载插件，但是下载插件的前提是电脑上必须由Node.js的环境。**其他用户不需要下载**

## 3.腾讯云Cos的申请

​		接下来介绍腾讯云Cos的申请，目前时间节点，腾讯云有活动，可以一元免费获取存储空间50G，有较高性能需求的用户可以自行购买。

腾讯云cos链接：https://cloud.tencent.com/act/pro/cos?fromSource=gwzcw.2572264.2572264.2572264&utm_medium=cpc&utm_id=gwzcw.2572264.2572264.2572264

==我们选择第一个即可==

![image-20220820112652607](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820112652607.png)

购买成功后：第一步我们进入控制台，创建一个存储桶根据相应的规则选择即可

![image-20220820112927108](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820112927108.png)

拥有一个存储桶后，下一步我们需要创建一个秘钥：

![image-20220820113218582](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820113218582.png)

这里我们新建一个秘钥：

![image-20220820113243230](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820113243230.png)

## 4.PicGo设置

![image-20220820113411184](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820113411184.png)

前三个选项我们为秘钥信息对应粘贴即可：

![image-20220820113513602](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820113513602.png)

后两个选项为存储桶信息：

![image-20220820113619004](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820113619004.png)配置完成后选择设置为默认图床，点击确定！

==在图床设置中选择腾讯云COS==

![image-20220820113807720](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820113807720.png)

接下来进行测试环节：

==将一张图片拖入到框中，然后我们查看相册，会有对应的图片，这样表明上传成功==

![image-20220820113939694](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820113939694.png)

## 5.Typora的设置

Typora我们需要下载1.0版本之后的，支持图床上传的版本：

 Typora下载安装破解教程链接：

点击文件->偏好设置—>图像

配置图中五个设置即可，picgo路径为个人的安装路径

![image-20220820114205660](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820114205660.png)

==这是我们将一张图片放入typora中图片的连接就会显示对应的网址==

![image-20220820114520833](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220820114520833.png)



#### 附件分享：

**个人gitee笔记**：[https://gitee.com/fanggaolei/learning-notes-warehouse](https://gitee.com/fanggaolei/learning-notes-warehouse)

==包含JavaWEB 大数据 算法 SQL Java等typora文档。附带对应的SSM项目和大数据项目，欢迎大家多多Starred==