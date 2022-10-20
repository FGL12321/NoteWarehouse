[TOC]



# Web项目部署到云服务器中

## 1.1工具准备

| 序号 | 工具         | 版本                                                |
| ---- | ------------ | --------------------------------------------------- |
| 1    | 阿里云服务器 | 1核1G CenterOS7.3                                   |
| 2    | Xshell       | 7.0                                                 |
| 3    | Xftp         | 7.0                                                 |
| 4    | Navicat      | 15.0                                                |
| 5    | JDK          | Linux版本：jdk-8u212-linux-x64.tar.gz               |
| 6    | Mysql        | Linux版本：mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar |
| 7    | Tomcat       | Linux版本：apache-tomcat-8.5.75.tar.gz              |

## 1.2安装JDK

**（1）将文件放入/opt/software目录下**

```
 jdk-8u212-linux-x64.tar.gz
```

**（2）将JDK和Hadoop两个文件分别解压**

```
#卸载虚拟机自带的java
rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps

tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/
```

**（3）添加JDK的环境变量**

```
sudo vim /etc/profile.d/my_env.sh
```

```
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_212
export PATH=$PATH:$JAVA_HOME/bin

最后让文件生效：
source /etc/profile
```

**（4）对虚拟机进行重启并验证JDK是否安装成功**

```
java -version
```

## 1.3安装Tomcat

**（1）上传Tomcat文件包到/opt/softwere**

```
apache-tomcat-8.5.75 
```

**（2）解压Tomacat**

```
tar -zxvf apache-tomcat-8.5.75 
mv  apache-tomcat-8.5.75 /opt/module
```

## 1.4安装MySql

**1）检查当前系统是否安装过 MySQL**

```
[atguigu@hadoop102 ~]$ rpm -qa|grep mariadb
mariadb-libs-5.5.56-2.el7.x86_64 
//如果存在通过如下命令卸载
[atguigu @hadoop102 ~]$ sudo rpm -e --nodeps mariadb-libs
```

**2）将 MySQL 安装包拷贝到/opt/software 目录下**

```
[atguigu @hadoop102 software]# ll
总用量 528384
-rw-r--r--. 1 root root 609556480 3 月 21 15:41 mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
```

**3）解压 MySQL 安装包**

```
[atguigu @hadoop102 software]# tar -xf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
```

**4）在安装目录下执行 rpm 安装**

```
[atguigu @hadoop102 software]$ 
sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
```

​		==过程中出现报错则说明需要安装依赖，通过 yum 安装缺少的依赖,然后重新安装 mysql-community-server-5.7.28-1.el7.x86_64 即可==cd

```
[atguigu@hadoop102 software] yum install -y libaio
```

**5）删除/etc/my.cnf 文件中 datadir 指向的目录下的所有内容,如果有内容的情况下:**

 **查看 datadir 的值：** 

```
[mysqld]
datadir=/var/lib/mysql
```

**删除/var/lib/mysql 目录下的所有内容（一般没有任何东西）:**

```
[atguigu @hadoop102 mysql]# cd /var/lib/mysql
[atguigu @hadoop102 mysql]# sudo rm -rf ./* //注意执行命令的位置
```

**6）初始化数据库**

```
[atguigu @hadoop102 opt]$ sudo mysqld --initialize --user=mysql
```

**7）查看临时生成的 root 用户的密码**

```
[atguigu @hadoop102 opt]$ sudo cat /var/log/mysqld.log       jZu%i>4hD
```

![1660702382426](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660702382426.png)

**8）启动 MySQL 服务**

```
[atguigu @hadoop102 opt]$ sudo systemctl start mysqld
```

**9）登录 MySQL 数据库**

```
[atguigu @hadoop102 opt]$ mysql -uroot -p
Enter password: 输入临时生成的密码
```

**10）必须先修改 root 用户的密码,否则执行其他的操作会报错**

```
mysql> set password = password("fgl123");
```

**11）修改 mysql 库下的 user 表中的 root 用户允许任意 ip 连接**

```
mysql> update mysql.user set host='%' where user='root';
mysql> flush privileges;
```

**12）使用本机Navicat连接远程数据库**

![1660702611954](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660702611954.png)

## 1.5配置防火墙

**1.查看防火墙服务状态**

```
systemctl status firewalld
```

![1660703030846](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660703030846.png)

**2.如果没有开启可进行如下操作**

```
#开启服务
service firewalld start
#关闭服务
service firewalld stop
#重启服务
service firewalld restart
```

**3.查看防火墙状态并开启防火墙**

```
#查看防火墙状态
firewall-cmd --state
#开启防火墙
service mysqld start
```

![1660703092637](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660703092637.png)

**4.查看防护墙规则**

```
firewall-cmd --list-all
```

![1660703933950](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660703933950.png)

**5.如果没有开放这些端口可使用如下命令开放**

```
#开放80端口
firewall-cmd --permanent --add-port=80/tcp
#开放8080端口
firewall-cmd --permanent --add-port=8080/tcp
#开放3306端口
firewall-cmd --permanent --add-port=3306/tcp

#移除端口命令
firewall-cmd --permanent --remove-port=3306/tcp
```

**6.开通完成后重启防火墙**

```
firewall-cmd --reload
```

**7.重新查看端口是否开启**

```
firewall-cmd --list-all
```

## 1.6配置云服务器安全组

**以阿里云为例**

![1660704395013](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660704395013.png)



![1660704437061](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660704437061.png)

![1660704460205](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660704460205.png)

![1660704509610](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660704509610.png)

==本机已经开启所以手动添加为灰色==

## 1.7Tomcat配置与启动

**1.将war包放到tomcat目录下的webapp中**

**2.文件配置修改con中的server.xml配置文件**

```
#进入conf目录
cd conf

#修改配置文件
vim server.xml

#将8080端口该为80端口

#按照个人war包在系统中的地址进行配置
<Context docBase="/opt/module/apache-tomcat-8.5.75/webapps/ssmhe" path="" reloadable="false"/>
```

![1660704767005](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660704767005.png)

![1660704993900](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660704993900.png)

![1660704848608](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660704848608.png)

**3.启动Tomcat**

```
#进入bin目录

#开启tomcat（等待20秒）
./startup.sh

#关闭tomcat
./shutdown.sh

#重新开启
./startup.sh
```

**4.通过云服务器公网IP在浏览器中进行访问**

如： 47.94.139.13 

==可通过购买域名与公网IP进行配置从而让用户使用域名进行访问==xs

## 1.8补充知识

### 8.8.1如何将项目打war包

**1.在IDEA中打开Project Structure**

![1660705598516](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705598516.png)

**2.选择**

![1660705637100](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705637100.png)

**3.点击+号**

![1660705666763](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705666763.png)

**4.选择**

![1660705700030](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705700030.png)

**5.配置并打包**

![1660705778126](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705778126.png)

![1660705807161](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705807161.png)

![1660705831419](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705831419.png)

### 8.8.2如何在本地Tomcat运行war包

**1.将war包放入本地Tomcat目录中的webapp下**

![1660706036288](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660706036288.png)

**2.在tomcat  bin目录下打开终端输入.\startup.bat**

![1660706169703](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660706169703.png)

**3.弹出运行框正常运行即可访问web项目**

![1660706234298](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660706234298.png)

**4.关闭Tomcat**

![1660706297352](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660706297352.png)

