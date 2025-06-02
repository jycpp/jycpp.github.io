---
title: openEuler 22.03安装记录
author: [Jet Yan]
date: 2024-05-06 17:27:20
version: 1.0
comments: true
tags:
- MacOS
- Git操作
- XCode
- 版本控制
- Gitee
- 项目管理
- 代码提交
- Dockerfile
categories:
- 软件开发
- 运维管理
---



## 1 系统安装及初始化




网络配置使用： nmcli

查看IP： ip addr

安装ifconfig没有成功： yum install ifconfig，提示安装包没有。

yum源的网址： /etc/yum.repos.d/openEuler.repo



### **安装Xfce**

由于安装的时候采用了最小安装，所以这里需要补充安装一个IDE。

使用SecureCRT连接服务器，按照官方文档步骤安装IDE： Xfce。

Xfce的配置指南： https://docs.openeuler.org/zh/docs/22.03\_LTS\_SP2/docs/desktop/Xfce\_userguide.html

其中设置网络：（右键点击）

![](https://s2.loli.net/2025/02/11/s6yRzC5bjlIeEMP.png)



#### a.设置固定IP

设置了临时固定IP： 192.168.71.71



#### b.去掉自动锁屏

去掉自动锁屏的功能：

运行命令： xfce4-screensaver-preferences

![](https://s2.loli.net/2025/02/11/vEit3pldwkK1agG.png)



### **安装Firefox：**

（安装工具dnf，相当于yum）安装完成后在Xfce中指定网络浏览器为Firefox。

```plain&#x20;text
sudo dnf install firefox
```



### dnf管理软件包



【https://docs.openeuler.org/zh/docs/21.03/docs/Administration/%E4%BD%BF%E7%94%A8DNF%E7%AE%A1%E7%90%86%E8%BD%AF%E4%BB%B6%E5%8C%85.html#%E6%90%9C%E7%B4%A2%E8%BD%AF%E4%BB%B6%E5%8C%85】



```bash
dnf list php  #检查到PHP的版本为8.0
dnf search httpd

dnf list all #显示所有可用的软件包，大约将近3万个
dnf check-update 检查安装了的rpm的最新版本

dnf info httpd
dnf info ruby

#升级软件包
dnf update ruby
```



### ifconfig：ip addr

没有ifconfig，使用命令  ip addr 替代



**netstat命令：如果没有，则安装**

dnf install net-tools



### firewall-cmd命令：



```bash
# 允许http和https两项服务通过防火墙，分别对应80和443端口
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
【success】
# 重新加载刚才的防火墙配置
firewall-cmd --reload
【success】

#以下直接增加端口的方法，没有试过
#firewall-cmd --add-port=443/tcp
# 加入一个端口到区域：
firewall-cmd --add-port=19069/tcp

firewall-cmd --zone=public --add-port=7081/tcp --permanent
firewall-cmd --zone=public --add-port=7051/tcp --permanent
firewall-cmd –reload

查看public zone区域的所有端口：
firewall-cmd --zone=public --list-ports

```



### DDNS-GO



【25.2.7. 动态更新域名的IP地址】 DDNS-GO

https://registry.hub.docker.com/r/jeessy/ddns-go/

![](https://s2.loli.net/2025/02/11/buz2fGndwhZ1spa.png)





## 2 搭建Web服务器



主要的参考文档：

（1）openEuler搭建Web服务器：https://docs.openeuler.org/zh/docs/22.03\_LTS\_SP2/docs/Administration/%E6%90%AD%E5%BB%BAweb%E6%9C%8D%E5%8A%A1%E5%99%A8.html

（2）搭建FTP和数据库服务器

https://docs.openeuler.org/zh/docs/22.03\_LTS\_SP2/docs/Administration/%E6%90%AD%E5%BB%BAFTP%E6%9C%8D%E5%8A%A1%E5%99%A8.html

https://docs.openeuler.org/zh/docs/22.03\_LTS\_SP2/docs/Administration/%E6%90%AD%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%BA%93%E6%9C%8D%E5%8A%A1%E5%99%A8.html



### 2.0 **创建目录**

```bash

mkdir /data

mkdir /data/website
mkdir /data/website/default
mkdir /data/mysql
mkdir /data/httpd_logs

# chown: 无效的用户: “apache:apache”
chown -R apache:apache /data/website/
chown -R apache:apache /data/httpd_logs/
chown -R mysql:mysql /data/mysql/
```


### 2.1 安装Apache和HTTP



![](https://s2.loli.net/2025/02/11/mNvgdycFbYrAR5U.png)



```bash
# 组合安装
dnf -y install httpd php
# Apache的依赖
dnf -y install mod_ssl mod_perl
# PHP的依赖
dnf -y install php-gd php-xml php-mbstring php-ldap php-pear php-mysqli
```



#### 日常管理命令


```bash
# 验证httpd服务是否正在运行
systemctl is-active httpd

# 启动并运行httpd服务，命令如下：
systemctl start httpd
systemctl stop httpd

# 如果希望防止服务在系统开机阶段自动开启，命令和回显如下：
systemctl disable httpd
Removed /etc/systemd/system/multi-user.target.wants/httpd.service.

# 重启服务有三种方式：
systemctl restart httpd  #硬重启
systemctl reload httpd  #软重启（只更新配置，不重启进程，但是中断任务）
apachectl graceful   #软更新（只更新配置，不重启进程，不中断任务）

# 检查配置文件可能出现的语法错误
apachectl configtest

# 因为Apache通过php-fpm执行PHP文件，因此必要时候需要重启fpm
# 例如遇到 fastcgi错误
systemctl restart php-fpm



```



#### 防火墙相关的命令

```bash
# 允许http和https两项服务通过防火墙，分别对应80和443端口
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
【success】
# 重新加载刚才的防火墙配置
firewall-cmd --reload
【success】

#以下直接增加端口的方法，没有试过
#firewall-cmd --add-port=443/tcp
```



#### 配置Apache



apache配置文件存放的位置：vi /etc/httpd/conf/httpd.conf

（1）在Include conf.d/\*.conf下面增加 Include vhost.conf

（2）将 ServerName 注释放开，可以改为 www.longmix.com:80

（3）放开 NameVirtualHost \*:80 的注释  【CentOS 7 下已经没有这个选项了。】【2019-03-24 CentOS 7 中没有NameVirtualHost】

（4）增加：LogFormat "%V:%p %h %l %u %t \\"%r\\" %>s %O \\"%{Referer}i\\" \\"%{User-Agent}i\\"" vhost\_combined

（5）这里有两条语句，添加到httpd.conf文件中，以避免操作系统和Apache版本暴露：

ServerSignature Off

ServerTokens Prod

（6）关联PHP的设置在/conf.d/php.conf中，以及  /conf.modules.d/10-php.conf

php配置文件存放的位置：/etc/php.ini 和/etc/php.d里面的文件夹中

（7）组织vhost.conf文件的内容

```bash
# vi /etc/httpd/vhost.conf

<Directory "/data/website/default/">
    Options FollowSymLinks Includes
    AllowOverride All
    Require all granted
</Directory>

<VirtualHost *:80>
    ServerAdmin help@abot.cn
    DocumentRoot "/data/website/default"
    ServerName default.abot.cn
    # php_admin_value open_basedir ".:/tmp:/data/website/default"
    ErrorLog "|/usr/sbin/rotatelogs /data/httpd_logs/dummy-default-error-%Y_%m_%d.log 86400 480"
    CustomLog "|/usr/sbin/rotatelogs /data/httpd_logs/dummy-default-access-%Y_%m_%d.log 86400 480" vhost_combined
</VirtualHost>


```



![](https://s2.loli.net/2025/02/11/tiobhZXUgNduLJ6.png)



![](https://s2.loli.net/2025/02/11/DUVYKEv5yZ4Osd3.png)





#### 配置Apache遇到的问题



发现打开网页错报，查看日志发现以下问题。



```bash
[root@localhost default]# find / -name error_log
/var/log/httpd/error_log
/sys/kernel/tracing/error_log
/sys/kernel/debug/tracing/error_log

[root@localhost default]# tail -f /var/log/httpd/error_log 
```



（1）找不到retatelogs这个程序

Could not open log file '/data/httpd\_logs/dummy-default-access-2024\_05\_26.log' (Permission denied)

AH00106: piped log program '/usr/sbin/rotatelogs /data/httpd\_logs/dummy-default-access-%Y\_%m\_%d.log 86400 480' failed unexpectedly



去掉vhost中写滚动日志记录的设置：

“CustomLog "|/usr/sbin/rotatelogs /data/httpd\_logs/dummy-default-access-%Y\_%m\_%d.log 86400 480" vhost\_combined”



（2）权限不够

\[Sun May 26 15:46:49.898615 2024] \[core:notice] \[pid 4006:tid 4006] AH00094: Command line: '/usr/sbin/httpd -D FOREGROUND'

\[Sun May 26 15:46:58.563494 2024] \[core:error] \[pid 4011:tid 4167] (13)Permission denied: \[client 192.168.71.10:13625] AH00035: access to /zzz.html denied (filesystem path '/data/website/default/zzz.html') because search permissions are missing on a component of the path



需要关闭SELinux，否则会报这个错误。

“vi /etc/selinux/config”，修改配置。

或者命令行执行“setenforce 0”。



修改了以上两个问题后，两个测试的网页都可以正常访问：

http://192.168.71.71/zzz.html

http://192.168.71.71/zzz.php





#### 配置PHP



> 常用的配置文件：
>
> vi /etc/httpd/conf.d/php.conf&#x20;
>
> vi /etc/php.in
>
> vi /etc/php-fpm.conf
>
> vi /etc/php-fpm.d/www.conf



PHP配置文件：vi /etc/php.ini

**设置open\_basedir参数：**

open\_basedir = .

表示PHP只能打开VirtualHost设置的目录，不能再打开上一级目录。

《防止php木马跨站设置》

【2020-06-15 CentOS 8之后，php7采用fpm方式与apache集成，父级目录已经没有实际意义，所以不需要再设置。】



**限制文件的大小**

post\_max\_size = 300M  设置POST的最大值

upload\_max\_filesize = 200M



2016-3-15

**关闭php X-Powered-By 信息**



thinkphp 清除 X-Powered-By: ThinkPHP



找到文件，

ThinkPHP/Lib/Think/Core/View.class.php。

搜索到一下代码屏蔽即可。

header('X-Powered-By:ThinkPHP');





**去掉PHP版本号**

PHP清除X-Powered-By: PHP/5.2.4



设置php.ini ，因此PHP版本号

expose\_php = Off



**禁止eval等敏感函数被黑客执行**

disable\_functions = eval,exec

【24.5.26.】 访问PHP文件可能会报错“No input file specified.”



**安装bcsub等函数的扩展**



dnf -y install php-bcmath



**CentOS下安装mcrypt的系统扩展**

~~dnf -y install libmcrypt libmcrypt-devel mcrypt mhash~~

dnf -y install libmcrypt libmcrypt-devel mhash



安装mcrypt的PHP扩展包  支持函数 mcrypt\_module\_open&#x20;

~~dnf -y install php-mcrypt~~


【**CentOS 8.0 安装 json_encode**】

~~dnf install php-json -y~~



dnf list installed | grep php

php -v



[root@VM-0-16-centos /]# php -v

PHP 7.2.24 (cli) (built: Oct 22 2019 08:28:36) ( NTS )

Copyright (c) 1997-2018 The PHP Group

Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies


**PHP7.4安装zip包**

问题来源：在 CMS控制台，导出所有粉丝遇到错误

Error: Class 'ZipArchive' not found in /data/website/weiduke\_cms/Weiduke/Lib/ORG/PHPExcel/PHPExcel/Writer/Excel2007.php:227

对应网址为：

http://cms.weiduke.com/User/Wechat\_group/index/setting\_type/export\_excel/all/1.shtml

http://yanyubao.tseo.cn/Supplier/Login/show\_yanyubao\_module\_list\_to\_excel.html

查看PHP的版本：

\[root@VM-0-2-centos tseo\_app]# php -v

查看可用的安装软件：

\[root@VM-0-2-centos tseo\_app]# dnf search php| grep zip

执行安装命令：

\[root@VM-0-2-centos tseo\_app]#~~ yum install -y php74-php-pecl-zip.x86\_64~~

**dnf install -y php-pecl-zip.x86\_64**

查看安装后的包：

\[root@VM-0-2-centos tseo\_app]# **php -m**

没有找到zip，安装之后，phpinfo()没有看到。



继续搜索资料后，发现是以上的安装方法不对，使用以下方法：

~~dnf install -y php-zip~~

安装后会自动创建30-zip.ini文件，如果没有创建，则可以增加40-zip.ini到目录/etc/php.d/，内容如下：



; Enable ZIP extension module

extension=zip.so



重启Apache，写以下代码测试：

var\_dump(class\_exists('ZipArchive'));

应该返回true。



![](https://s2.loli.net/2025/02/11/4DkeKarRqLTHPOB.png)





### 2.2 安装MySQL



```bash
# 可以直接安装
dnf install mysql-server
```



![](https://s2.loli.net/2025/02/11/gInzK8xamqslXtu.png)



#### 配置mysql



mv /etc/my.cnf /etc/my.cnf.old



vi /etc/my.cnf　 ← 编辑MySQL的配置文件

```bash
[mysqld]
datadir=/data/mysql
log-error=/data/mysql/mysqld.log
socket=/var/lib/mysql/mysql.sock
user=mysql
#Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

#reset root password
#skip-grant-tables


[mysqld_safe]
log-error=/data/mysql/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[mysql]
```



**给目录授权：**

chown -R mysql:mysql /data/mysql/



#### 启动MySQL服务：

```bash
#启动服务
systemctl start mysqld

#查看服务状态
systemctl status mysqld

#将MySQL和Apache都设置为开机启动
systemctl enable mysqld
systemctl enable httpd

#防火墙放行

```



#### 防火墙相关的命令

```bash
# 允许httpd通过防火墙
firewall-cmd --add-service=mysql --permanent
【success】
# 重新加载刚才的防火墙配置
firewall-cmd --reload
【success】

```



#### 设置root远程访问



本地访问时候root没有密码。


```bash
mysql> CREATE USER 'root'@'%' IDENTIFIED BY '你的密码';
Query OK, 0 rows affected (0.48 sec)
mysql> grant all privileges on *.* to 'root'@'%';
Query OK, 0 rows affected (0.48 sec)
mysql> FLUSH PRIVILEGES;
```



以上步骤可以成功设置，如果有问题，可以再参考以下内容：

设置生成密码的规则，mysql8 之前的版本中加密规则是mysql\_native\_password,而在mysql8之后,加密规则是caching\_sha2\_password

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password'; #修改加密规则 
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; #更新一下用户的密码 
FLUSH PRIVILEGES; #刷新权限 
```



### 2.3 安装vsftpd





安装

dnf install vsftpd -y



安装pam，用户设置用户列表

~~dnf install pam -y  （已经安装了）~~



#### 日常维护指令

```bash
# 指令跟Apache和MySQL类似
systemctl start vsftpd

#开机启动
systemctl enable vsftpd

# 在CentOS中，也可以使用这些命令重启服务
#service vsftpd restart
#chkconfig vsftpd on

# 防火墙 （FTP标准的21端口）
firewall-cmd --add-service=ftp --permanent
# 加入一个端口到区域：
firewall-cmd --add-port=19069/tcp
#【】success
firewall-cmd --reload
#【】success
# setsebool -P ftpd_full_access on

```



#### 设置vsftpd



mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.old



实际设置的内容



**vi /etc/vsftpd/vsftpd.conf**

```bash
listen=YES
write_enable=YES
nopriv_user=apache
anonymous_enable=NO
local_enable=YES
chroot_local_user=YES
chroot_list_enable=NO
dirmessage_enable=YES
xferlog_enable=YES
xferlog_file=/data/httpd_logs/xferlog
xferlog_std_format=YES
idle_session_timeout=600
data_connection_timeout=120
guest_enable=YES
guest_username=apache
pam_service_name=/etc/pam.d/vsftpd
user_config_dir=/etc/vsftpd/vuser_conf
listen_port=21
pasv_enable=YES
pasv_min_port=3300
pasv_max_port=3310
pasv_promiscuous=YES
ftpd_banner=Welcome to http://www.abot.cn FTP service
allow_writeable_chroot=YES
```



\-----------以下是默认的被动模式的端口-------------

pasv\_min\_port=3300            #设置被动模式的端口范围

pasv\_max\_port=3310            #设置被动模式的端口范围



```
mv /etc/pam.d/vsftpd /etc/pam.d/vsftpd.old
```


然后到&#x20;

**vi /etc/pam.d/vsftpd** ，只保留以下内容

```bash
#%PAM-1.0
auth    required     /lib64/security/pam_userdb.so    db=/etc/vsftpd/vsftpd_vuser
account    sufficient     /lib64/security/pam_userdb.so    db=/etc/vsftpd/vsftpd_vuser
```



#### 配置vsftpd虚拟用户

先创建本地用户：

```bash
#创建本地用户（-d指定家目录，-s指定登陆shell）
#useradd -d /var/ftproot -s /sbin/nologin vmftp

#但是我们这里已经指定了虚拟用户为apache，见配置文件“/etc/vsftpd/vsftpd.conf”
#其中的项目  guest_username=apache
```



使用gdbmtool创建虚拟用户列表：

```bash
#创建数据库的命令
gdbmtool
#创建数据库的位置
open /etc/vsftpd/vsftpd_vuser.pag
#创建一个用户名为ftp，密码为123
store ftp 123
#创建一个用户名为adm，密码为456
store adm 456
#查看创建的用户
list
#退出
quit
```









## 3 安装docker+Gitlab+Jenkins


### 3.1 总结



#### 3.1.1 私有docker镜像仓库



```bash
启动
docker start registry

运行后，浏览器访问：
http://192.168.0.83:5000/v2/_catalog

【没有做公网映射】
```



#### 3.1.2 Gitlab服务



```bash
启动
docker start gitlab

运行后浏览器访问：
http://192.168.0.83:7080/


```



#### 3.1.4 Jenkins服务



```bash
http://192.168.0.83:7081
```





### 3.2 问题和知识点



#### 遇到yum install -y yum-utils报错

```plain&#x20;text
openEule安装docker 遇到yum install -y yum-utils报错，No match for argument：Unable to find a match：

下载可用的.repo文件：
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
【#注 wget -O … （此处为大写的英文字母O）】

输入带版本的安装命令就可以按了
sudo yum install -y --releasever=8 yum-utils

```



#### 安装netstat



跟上面通用的方法，带版本安装

yum install --releasever=8 netstat

但是没有效果，使用：

yum install net-tools

安装成功。



查看端口号：

netstat -anotp





#### 安装最新版本的docker：


```plain&#x20;text
先增加docker官方仓库：

yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
cd /etc/yum.repos.d/
vi docker-ce.repo

通过 :1,$s/\$releasever/7/g  批量替换。


先查询一下
yum list docker-ce --showduplicates|sort -r
看有什么可以安装的版本


yum -y install docker-ce-24.0.2

遇到错误：
try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages
使用命令：
dnf install docker-ce docker-ce-cli containerd.io --nobest

最后启动docker服务
systemctl start docker

把服务改为开机自动启动
systemctl enable docker

检查一下安装的docker版本
docker --version

华为openEuler-22.03-LTS-SP1 对应 centos什么版本？
华为openEuler-22.03-LTS-SP1 对应的 CentOS 版本是 CentOS 7.91


出现以错误，根据提示在命令结尾加上–allowerasing或–nobest后再次执行即可

yum install -y docker-ce --nobest
yum install -y docker-ce --allowerasing
1
2
启动docker并设置开机自启

systemctl start docker && systemctl enable docker
1
查看Docker是否安装成功

docker version
```



#### 卸载docker



```plain&#x20;text

查询docker的包：
rpm -qa | grep docker

dnf remove docker-ce
dnf remove docker-ce-cli

再查询，没有了。
```


#### docker:使用阿里云的镜像加速器：

如果Linux系统是在阿里云里面的，可以使用这种方式加速。

![](https://s2.loli.net/2025/02/11/6CXTjND1aGP798i.png)



#### docker:国内镜像源加速



官方的镜像为：https://hub.docker.com/r/minio/minio#test-using-minio-client-mc



https://cloud.tencent.com/developer/article/2483548



vi /etc/docker/daemon.json

```json
{
  "insecure-registries":["192.168.0.83:5000"],
  "dns": ["8.8.8.8", "8.8.4.4"],
  "registry-mirrors": [
    "https://docker.hpcloud.cloud",
    "https://docker.m.daocloud.io",
    "https://docker.unsee.tech",
    "https://docker.1panel.live",
    "http://mirrors.ustc.edu.cn",
    "https://docker.chenby.cn",
    "http://mirror.azure.cn",
    "https://dockerpull.org",
    "https://dockerhub.icu",
    "https://hub.rat.dev",
    "https://proxy.1panel.live",
    "https://docker.1panel.top",
    "https://docker.m.daocloud.io",
    "https://docker.1ms.run",
    "https://docker.ketches.cn",
    "https://registry.hub.docker.com","http://hub-mirror.c.163.com","https://docker.mirrors.ustc.edu.cn","https://registry.docker-cn.com"
  ]
}
```



运行后，执行

systemctl daemon-reload

和

systemctl restart docker





#### docker:搭建私有仓库（registry）

![](https://s2.loli.net/2025/02/11/CqMZ8xj5tGLgnWu.png)



```plain&#x20;text
拉取源镜像：
docker pull registry
查看本地镜像：
docker images

从镜像创建容器（此镜像为本地源镜像，特殊的镜像）：
docker run -d -p 5000:5000 registry

按照上面的流程为docker增加本地源镜像 192.168.0.83:5000

查看本地创建的容器：
docker ps -a

启动容器：（如下图）
docker start 330e77a24bc4

运行后，浏览器访问：
http://192.168.0.83:5000/v2/_catalog
可以查看私有的镜像源，搭建完毕。

```



![](https://s2.loli.net/2025/02/11/w31BrcaoUIVpQgR.png)

![](https://s2.loli.net/2025/02/11/npuVIeLdQ1YUHEa.png)



修改daemon.json，增加私有仓库：

```bash
"insecure-registries":["192.168.0.83:5000"],
```



可以通过push将镜像推送到私有仓库：

Docker push \[私有仓库地址包括端口]/\[镜像名称]:\[镜像版本]





#### docker:总结操作命令


查看镜像： docker images

查看容器： docker ps -a

运行容器： docker run \[一堆参数]  【docker run相当于执行了两步操作：将镜像放入容器中（docker create）,然后将容器启动，使之变成运行时容器（docker start）】

启动容器： docker start  \[容器ID或name]

查看容器运行日志： docker logs -f \[容器ID或name]

重启容器： docker restart  \[容器ID或name]

停止容器： docker stop  \[容器ID或name]

删除容器： docker rm  \[容器ID或name]

删除镜像： docker rmi \[镜像ID]

进入容器中执行命令： **docker container exec -it \[docker id] /bin/bash** ，执行结束后exit。

进入容器中执行命令： **docker run -it \[docker name] /bin/bash**

查看容器版本： docker version （服务器和客户端）， 或 docker -v，仅查看客户端的版本。

创建镜像： docker build -t \[docker的文件名]:\[docker的Tags] .   【最后的“.”代表有Dockerfile的目录。】



进入容器之后，可以通过evn查看环境变量（EVN）。



#### docker compose编排服务



【docker-compose.yaml】









#### 安装Gitlab（用docker）



```plain&#x20;text
#拉取镜像
docker pull gitlab/gitlab-ce

#查看本地镜像
docker images

#本机建立的3个目录
#为了gitlab容器通过挂载本机目录启动后可以映射到本机，然后后续就可以直接在本机查看和编辑了，不用再进容器操作
#配置文件
mkdir -p /data/gitlab/etc
#数据文件
mkdir -p /data/gitlab/data
#日志文件
mkdir -p /data/gitlab/1ogs

#启动容器【没有使用，因为当时这个镜像拉取总是失败，所以使用了下面的极狐的】
docker run --name='gitlab' -d \
 --publish 7043:443 --publish 7080:80 \
 -v /data/gitlab/etc:/etc/gitlab \
 -v /data/gitlab/data:/var/opt/gitlab \
 -v /data/git1ab/1ogs:/var/1og/gitlab \
 --restart always \
 gitlab/gitlab-ce:latest 
 
#查看启动日志
docker 1ogs -f gitlab

其中参数“--restart always \” 是让这个docker随着openEuler系统一起启动

```



以上拉取镜像总是被卡死，如下：

```bash
[root@localhost ~]# docker pull gitlab/gitlab-ce
Using default tag: latest
latest: Pulling from gitlab/gitlab-ce
6414378b6477: Pull complete 
a625401f465a: Pull complete 
e8907e8d964d: Pull complete 
fa153a8aca38: Pull complete 
e37d8f6ef4c4: Pull complete 
16d0e00c967b: Pull complete 
790773297c19: Pull complete 
54cf79629b55: Pull complete 
a390b2bab26f: Waiting 
^C
```



于是尝试使用极狐Gitlab：

https://gitlab.cn/resources/articles/09091723-86a3-4f63-86e9-feb77b0e4289

```bash
[root@localhost ~]# docker pull docker.888666222.xyz/library/gitlab/gitlab-ce
Using default tag: latest
Error response from daemon: Head "https://docker.888666222.xyz/v2/library/gitlab/gitlab-ce/manifests/latest": Get "https://auth.docker.io/token?scope=repository%3Alibrary%2Fgitlab%2Fgitlab-ce%3Apull&service=registry.docker.io": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
[root@localhost ~]# docker pull registry.gitlab.cn/omnibus/gitlab-jh:latest
latest: Pulling from omnibus/gitlab-jh
6414378b6477: Pull complete 
bc9b441a8dc7: Pull complete 
a2be2778da30: Pull complete 
063742421638: Pull complete 
cc88ff8054d4: Pull complete 
3b1e4d6d4683: Pull complete 
fb47474ef7a7: Pull complete 
d10127e76315: Pull complete 
2c99056d70c7: Pull complete 
Digest: sha256:5ae0cf69d01c8745c04fb1f993b11bf305cb492527b73a2828210ab9e5cb2b2f
Status: Downloaded newer image for registry.gitlab.cn/omnibus/gitlab-jh:latest
registry.gitlab.cn/omnibus/gitlab-jh:latest
```

对应的启动命令如下：

```bash
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --shm-size 256m \
  registry.gitlab.cn/omnibus/gitlab-jh:latest
```

跟对应的英文官方的比较：

```bash
docker run --name='gitlab' -d \
 --publish 7043:443 --publish 7080:80 \
 -v /data/gitlab/etc:/etc/gitlab \
 -v /data/gitlab/data:/var/opt/gitlab \
 -v /data/gitlab/1ogs:/var/1og/gitlab \
 gitlab/gitlab-ce:latest 


 
```



##### 启动极狐Gitlab

```bash
#综合后的启动命令 
docker run --name='gitlab' -d \
 --publish 7043:443 --publish 7080:80 --publish 7022:22 \
 -v /data/gitlab/etc:/etc/gitlab \
 -v /data/gitlab/data:/var/opt/gitlab \
 -v /data/gitlab/1ogs:/var/1og/gitlab \
 --shm-size 256m \
 --restart always \
 registry.gitlab.cn/omnibus/gitlab-jh:latest 
```



放入容器后，配置Gitlab，参考这篇文档：

https://developer.aliyun.com/article/922952



**访问的网址为：**

**http://192.168.0.83:7080/**

似乎不需要设置external url之类的参数。



在Apache中做公网映射：

```bash
<VirtualHost *:80>
    ServerName gitlab.test.tseo.cn
    ProxyPass / http://127.0.0.1:7080/
    ProxyPassReverse / http://127.0.0.1:7080/
    ErrorLog /data/httpd_logs/dummy-test.tseo.cn-error.log
    CustomLog /data/httpd_logs/dummy-test.tseo.cn-access.log vhost_combined
</VirtualHost>

```



```bash
[root@localhost gitlab]# find ./ -name init*
./etc/initial_root_password

[root@localhost gitlab]# ll etc/
总用量 204
-rw------- 1 root root 157735  1月 26 13:46 gitlab.rb
-rw------- 1 root root  16358  1月 26 13:46 gitlab-secrets.json
-rw------- 1 root root    749  1月 26 13:02 initial_root_password
-rw------- 1 root root    513  1月 26 13:02 ssh_host_ecdsa_key
-rw-r--r-- 1 root root    179  1月 26 13:02 ssh_host_ecdsa_key.pub
-rw------- 1 root root    411  1月 26 13:02 ssh_host_ed25519_key
-rw-r--r-- 1 root root     99  1月 26 13:02 ssh_host_ed25519_key.pub
-rw------- 1 root root   2602  1月 26 13:02 ssh_host_rsa_key
-rw-r--r-- 1 root root    571  1月 26 13:02 ssh_host_rsa_key.pub
drwxr-xr-x 2 root root   4096  1月 26 13:02 trusted-certs

[root@localhost gitlab]# vi /data/gitlab/etc/initial_root_password 
[root@localhost gitlab]# 
[root@localhost gitlab]# 
[root@localhost gitlab]# 
```



#### 安装Java 1.8开发环境


```bash

[root@localhost ~]# wget https://corretto.aws/downloads/latest/amazon-corretto-8-x64-linux-jdk.rpm


[root@localhost ~]# yum localinstall amazon-corretto-8-x64-linux-jdk.rpm

[root@localhost ~]# java -version
openjdk version "1.8.0_442"
OpenJDK Runtime Environment Corretto-8.442.06.1 (build 1.8.0_442-b06)
OpenJDK 64-Bit Server VM Corretto-8.442.06.1 (build 25.442-b06, mixed mode)
```



#### 安装Maven 3.6.3环境



1. 下载指定版本，解压。

2. 配置conf下的国内镜像源。

3. 使用绝对路径访问。





```bash
直接下载：

[root@localhost ~]# wget https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz

解压：
[root@localhost ~]# tar zxvf apache-maven-3.6.3-bin.tar.gz 

使用绝对路径：
[root@localhost bin]# /root/apache-maven-3.6.3/bin/mvn -v
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /root/apache-maven-3.6.3
Java version: 1.8.0_442, vendor: Amazon.com Inc., runtime: /usr/lib/jvm/java-1.8.0-amazon-corretto/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "5.10.0-247.0.0.149.oe2203sp3.x86_64", arch: "amd64", family: "unix"
[root@localhost bin]# 

```

下载完成后，在conf目录下为Maven设置阿里云镜像，过程同Windows 11。





#### 安装Jenkins



```bash
docker run \
  --name=jenkins \
  -u root \
  -d \
  -p 7081:8080 \
  -p 7051:50000 \
  -v /run/docker.sock:/var/run/docker.sock \
  -v /usr/bin/docker:/usr/bin/docker \
  -v /usr/lib/jvm/java-1.8.0-amazon-corretto:/usr/java/jdk1.8.0_181 \
  -v /root/apache-maven-3.6.3:/usr/local/maven \
  -v /root/maven_local_repository:/usr/local/maven_repository \
  -v /data/jenkins-data:/var/jenkins_home \
  --restart always \
  jenkins/jenkins:lts


以上参数，可以通过find查找：
[root@localhost ~]# find / -name docker.sock
/run/docker.sock
[root@localhost ~]# 
同样的，分别查找 docker  java  maven

其中：
-u root为指定root用户，是Jenkins的root用户，不是Linux的。



  
```



打开防火墙：

```bash
firewall-cmd --zone=public --add-port=7081/tcp --permanent
firewall-cmd --zone=public --add-port=7051/tcp --permanent
firewall-cmd –reload
```


进入容器中查看root的密码：

```bash

[root@localhost ~]# 
[root@localhost ~]# 
[root@localhost ~]# 
[root@localhost ~]# docker exec -it jenkins bash
root@8f2c9dfb22e1:/# cat /var/jenkins_home/secrets/initialAdminPassword
cb00dd69a5034e69a525766ac264bbd8
root@8f2c9dfb22e1:/# 
root@8f2c9dfb22e1:/# exit
exit
[root@localhost ~]# 
[root@localhost ~]# 
[root@localhost ~]# 


```



访问网址，登录Jenkins：

http://192.168.0.83:7081

http://127.0.0.1:7081



首次登录需要设置新增一个管理员用户，默认是admin，这里我们设置了root。


增加Apache的配置：

```bash
<VirtualHost *:80>
    ServerName jenkins.test.tseo.cn
    ProxyPass / http://127.0.0.1:7081/
    ProxyPassReverse / http://127.0.0.1:7081/
    ErrorLog /data/httpd_logs/dummy-test.tseo.cn-error.log
    CustomLog /data/httpd_logs/dummy-test.tseo.cn-access.log vhost_combined
</VirtualHost>
```


![](https://s2.loli.net/2025/02/11/YMxSR3Qkn4GiUlp.png)

#### k8s - Kubernetes 1.26.0 安装



![](https://s2.loli.net/2025/02/11/XuOHowBlqMpcS3h.png)



![](https://s2.loli.net/2025/02/11/xEBfcK9IPwdkgMX.png)





![](https://s2.loli.net/2025/02/11/TerQH91JtFDKgBl.png)

![](https://s2.loli.net/2025/02/11/PNip9dODL7safU5.png)








