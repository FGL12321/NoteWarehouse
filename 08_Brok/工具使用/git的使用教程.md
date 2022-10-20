# git与Github命令行使用教程

[TOC]

​		**所谓工欲善其事，必先利其器，git/github已经成为了开发者必不可少的一门技能，今天主要利用git和github介绍这项优质工具的使用！**

## 1.0 github的访问

​		很多初学者，在使用github的过程中都会出现访问慢，或者无法访问的以及克隆下载缓慢等情况，对此本人有专门的文章介绍了github文档访问教程：请点击获取[Github如何稳定-高效-安全访问](https://blog.csdn.net/m0_58022371/article/details/126558905)让你彻底摆脱github的苦恼！

## 2.0 多文件上传

==（已经配置好仓库ssh并完成了git的安装，并能够成功测试上传一些小文件）==

**首先建立一个仓库**

![image-20220828133511892](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220828133511892.png)

**需求说明：在本仓库存放一批大数据hadoop，hive，hbase等安装压缩包相当于一个网盘的使用。**

**一：将仓库克隆到本地：**

①复制克隆链接进行克隆

```
$ git clone 仓库链接
```

②将资源放入文件夹汇中

③上传文件

```
git add . #将文件添加到缓存中
git commit -m "8月28日" #添加说明
git push #上传
```

![image-20220828142847402](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220828142847402.png)

![image-20220828142827923](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220828142827923.png)

上传成功！

**二.将多个文件上传**

和上述方法相同！

![image-20220828143053538](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220828143053538.png)

![image-20220828143124662](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220828143124662.png)

上传成功！

## 3.0 不可隆上传

```java
git init //初始化仓库

git remote add origin 仓库网址   //将本地与新建仓库进行配对

git commit -m "添加说明"  

git push -u origin 分支名 //这样就把.gitattributes上传成功
```



## 4.0 常用命令汇总

```
git init //新建一个空的仓库

git status //查看状态

git add . //添加文件

git commit -m '注释' //提交添加的文件并备注说明

git remote add origin git@github.com:jinzhaogit/git.git //连接远程仓库

git push -u origin master //将本地仓库文件推送到远程仓库

git log 查看变更日志

git reset --hard //版本号前六位 回归到指定版本

git branch //查看分支

git branch newname //创建一个叫newname的分支

git checkout newname //切换到叫newname的分支上

git merge newname //把newname分支合并到当前分支上

git pull origin master //将master分支上的内容拉到本地上
```

## 5.0 优秀教程分享

1.[廖雪峰 - Git 教程](https://www.liaoxuefeng.com/wiki/896043488029600)      [访问量: 29656109033，新手必看]


