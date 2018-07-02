---
layout: post
category: "linux"
title:  "RabbitMQ集群搭建"
tags: [linux, RabbitMQ]
---

### RabbitMQ集群搭建

最近看了一下RabbitMQ的相关，准备搭建一个RabbitMQ集群，尝试集群部分的操作。



#### 安装准备

本例以两个节点为例，搭建一个多机多节点的RabbitMQ集群，在部署集群之前需要在每台机器上正确安装RabbitMQ，另外，RabbitMQ集群对延迟非常敏感，应只在本地局域网中使用。



#### 配置hosts文件

编辑/etc/hosts文件，添加如下信息：  
 	10.110.8.34   node1    
 	10.110.8.38   node2    
如果需要更改节点名称，可以编辑/etc/rabbitmq/rabbitmq-env.conf 文件，添加     

	NODENAME=rabbit@node1


#### 编辑cookie文件

编辑 RabbitMQ 的 cookie 文件，以确保各个节点的cookie 文件使用的是同一个值。 可以读取 node1 节点的 cookie 值， 然后将其复制到 node2中。 cookie 文件 默认 路径 为/ var/ lib/ rabbitmq/. erlang. cookie 或者$ HOME/. erlang. cookie。 cookie 相当于密钥令牌，集群中的 RabbitMQ 节点需要通过交换密钥令牌以获得相互认证。



#### 配置集群

通过 rabbitmqctl 工具配置；
先启动node1和node2节点的RabbitMQ服务
![启动服务]({{"/assets/linux/20180702/1.png" | absolute_url}})

这样两个节点目前是以独立节点存在的单个集群
rabbitmqctl cluster_status 查看目前两个节点的状态   
node1  
![节点状态]({{"/assets/linux/20180702/2.png" | absolute_url}})

node2  
![节点状态]({{"/assets/linux/20180702/3.png" | absolute_url}})

接下来，把node2加入node1组建一个集群，步骤如下：  
在node2节点执行  
rabbitmqctl stop_app  
rabbitmqctl reset  
rabbitmqctl join_cluster rabbit@node1  
rabbitmqctl start_app  
![node2执行]({{"/assets/linux/20180702/4.png" | absolute_url}})

rabbitmqctl cluster_status 查看目前节点的集群信息，如下：  
![集群状态]({{"/assets/linux/20180702/5.png" | absolute_url}})

集群已组建