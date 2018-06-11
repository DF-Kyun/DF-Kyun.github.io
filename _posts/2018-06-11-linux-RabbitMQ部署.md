---
layout: post
category: "linux"
title:  "RabbitMQ部署"
tags: [linux, RabbitMQ]
---

### RabbitMQ部署

项目产品最近准备使用RabbitMQ作为消息推送的中间件，今天在公司内网部署了一下，因为内网环境的原因，刚开始是想在线安装erlang，但是有几个rpm依赖包始终无法下载，最终改为源码包安装，在这里记录一下安装过程，中间有些问题可能并不是常见问题。

#### 安装准备
	1）erlang语言环境
		版本：20.3
	2）RabbitMQ
		版本：3.7.5 

#### erlang语言环境安装
官方下载安装包，官方地址：http://www.erlang.org/downloads，此次安装使用的版本是otp_src_20.3.tar.gz，下载之后上传到服务器

在上传目录解压   
	tar -xvf　otp_src_20.3.tar.gz   
安装依赖包   
	yum install -y gcc gcc-c++ unixODBC-devel openssl-devel ncurses-devel perl  
检查环境、设置安装位置   
	./configure --prefix=/usr/local/erlang --without-javac  
安装   
	make && make install  

配置环境变量   
	vi /etc/profile  
	添加  
	export PATH=$PATH:/usr/local/erlang/bin  
	保存，执行 source /etc/profile  

测试erlang环境  
	在命令行中输入erl，出现erlang环境，则安装成功  

问题解决：   
checking for perl... no_perl   
configure: error: Perl is required to generate v2 to v1 mib converter script   
configure: error: /bin/sh '/root/software/otp_src_17.1/lib/snmp/./configure' failed for snmp/.   
configure: error: /bin/sh '/root/software/otp_src_17.1/lib/configure' failed for lib   
 
\#如上，提示错误，解决方法：安装Perl   
[root@localhost otp_src_17.1]# yum install perl



#### RabbitMQ安装
官方下载安装包，官方地址：https://www.rabbitmq.com/install-rpm.html，本次使用的版本为rabbitmq-server-3.7.5-1.el7.noarch.rpm，下载之后上传服务器

rpm包安装  
rpm -ivh ./rabbitmq-server-3.7.5-1.el7.noarch.rpm  

安装会报错 有依赖关系  
socat is needed by rabbitmq-server-3.7.5-1.el7.noarch   

解决方法：   

yum -y install socat

此时会报错没有socat包或是找不到socat包，解决方法安装centos的epel的扩展源

yum -y install epel-release

之后执行yum -y install socat

重新安装 socat

接着安装rpm -ivh rabbitmq-server-3.7.5-1.el7.noarch.rpm 安装会报错，有依赖关系
![安装报错]({{"/assets/linux/20180611/1.png" | absolute_url}})

解决方法不安装依赖关系

rpm -i --nodeps rabbitmq-server-3.7.5-1.el7.noarch.rpm

生成配置文件

cp /usr/share/doc/rabbitmq-server-3.7.5/rabbitmq.config.example /etc/rabbitmq/rabbitmq.config

启动rabbitmq

service rabbitmq-server start

报错，启动失败
![启动报错]({{"/assets/linux/20180611/2.png" | absolute_url}})

查看错误详细信息如下：
![错误信息]({{"/assets/linux/20180611/3.png" | absolute_url}})

通过上面描述中，发现是rabbitmq-server文件第85没有找到erlang，解决方法：在rabbitmq-server文件第85行处添加erlang的环境变量
![环境变量]({{"/assets/linux/20180611/4.png" | absolute_url}})

再次启动，启动成功
![启动成功]({{"/assets/linux/20180611/5.png" | absolute_url}})

然后开启管理页面插件

rabbitmq-plugins enable rabbitmq_management

添加管理员账号密码，默认的管理员用户密码为（guest/guest）

rabbitmqctl add_user rabbitadmin 123456

分配用户标签 

rabbitmqctl set_user_tags rabbitadmin administrator

创建和赋角色完成后查看并确认

rabbitmqctl list_users
\#Listing users ...
\#admin	[administrator]
\#guest	[administrator]

登录rabbitmq管理界面

浏览器输入地址：http://服务器IP地址:15672/

用户名密码：rabbitadmin/123456

![访问页面]({{"/assets/linux/20180611/6.png" | absolute_url}})
