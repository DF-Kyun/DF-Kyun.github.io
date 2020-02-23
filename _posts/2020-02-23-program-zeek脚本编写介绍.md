---
layout: post
category: "program"
title:  "zeek脚本编写介绍"
tags: [program ,zeek]
---

最近学习了一下zeek，简单记录一下

## zeek介绍
Zeek是一个被动的开源网络流量分析器。它主要是一种安全监视器，可深入检查链接上的所有流量以查找可疑活动的迹象。使用Zeek最直接的好处是生成大量日志文件。这些日志不仅包括对网络上每个连接的全面记录，还包括应用程序层记录，例如所有HTTP会话及其请求的URI，密钥标头，MIME类型和服务器响应；带回复的DNS请求；SSL证书；SMTP会话的关键内容；以及更多。默认情况下，Zeek将所有这些信息写入结构合理的制表符分隔的日志文件中，这些文件适用于使用外部软件进行后处理。

另外，在名称上，3.0版本之前是叫bro，之后改为zeek，相关的命令也都有变动

## 脚本编写
基本命令

![]({{"/assets/program/20200223/1.png" | absolute_url}})

### 编写并通过bare模式执行脚本
Zeek的脚本语言是事件驱动的，Zeek中的脚本编写取决于Zeek在处理网络流量时生成的事件，通过这些事件更改数据结构的状态以及对所提供信息进行决策。

Zeek的核心作用是将事件放入有序的“事件队列”中，从而使事件处理程序可以按照“先到先得”的方式处理事件。实际上，这是Zeek的核心功能，因为如果没有编写脚本来对事件执行离散操作，则几乎没有可用的输出。对事件队列，生成的事件以及事件处理程序处理这些事件的方式的基本了解不仅是学习Zeek编写脚本的基础，而且还是了解Zeek本身的基础。

一个简单的例子

![]({{"/assets/program/20200223/2.png" | absolute_url}})

### 连接记录数据类型
**connection 类型**
在Zeek定义的所有事件中，绝大多数事件都传递了connection数据类型，连接记录本身是大量的嵌套数据类型，用于跟踪连接在其生命周期内的状态。

尽管Zeek能够进行数据包级别的处理，但其优势在于始发者与响应者之间的连接。因此，为连接生命周期的主要部分定义了一些事件，例如：
**new_connection**
**connection_timeout**
**connection_state_remove**

其中，对连接记录数据类型有最好的展示的事件是connection_state_remove。Zeek会在决定从内存中删除该事件之前生成该事件。

### 通过网络抓包的方式获取数据，然后执行脚本
通过抓包来获取数据，模拟解析网络数据，查看连接内容
![]({{"/assets/program/20200223/3.png" | absolute_url}})

![]({{"/assets/program/20200223/4.png" | absolute_url}})

![]({{"/assets/program/20200223/5.png" | absolute_url}})

### 记录源IP和访问IP，并计算两次访问间隔时间
![]({{"/assets/program/20200223/6.png" | absolute_url}})

### 自定义日志记录框架（Logging Framework）
![]({{"/assets/program/20200223/7.png" | absolute_url}})

首先，需要通知Zeek我们将通过向Log::ID枚举数添加值来添加另一个Log Stream 。在此脚本中，我们将值LOG追加到Log::ID枚举值，但是由于此值位于导出块中，Log::ID因此实际 将值追加到Factor::Log。
接下来，我们需要定义组成日志数据的名称和值对，并规定其格式。该脚本定义了一个新的记录数据类型，称为Info（实际上， Factor::Info）和四个字段，为addr和port。记录类型中的每个字段都包含&log 属性，在Log::write调用这些字段时应将这些字段传递给Logging Framework 。如果有任何不带&log属性的名称/值对，则将在记录期间仅忽略这些字段，但在变量的生命周期内仍然可用。
下一步是创建Log::create_stream，一个以 Log::ID和一条记录及日志文件作为其参数的日志记录流。在此示例中，我们调用 Log::create_stream方法并通过Factor::LOG、 Factor::Info记录和factor日志文件名作为参数。
从现在开始，向Log::write命令发出Log::ID且格式正确的Factor::Info记录，将生成日志条目。结果如下：
![]({{"/assets/program/20200223/8.png" | absolute_url}})

### 通知框架
要在Zeek中发出通知，只需要提供特定的Notice :: Type，然后调用NOTICE为其提供适当的Notice :: Info记录。通常，对NOTICE的调用仅包含Notice :: Type和简洁的消息。但是，在发出通知时，有更多选项，note :: Info中唯一必填的字段是note字段。
![]({{"/assets/program/20200223/9.png" | absolute_url}})

会在notice.log文件中记录通知信息
![]({{"/assets/program/20200223/10.png" | absolute_url}})

首先，脚本export块将LogNotice::Interesting_Hostname_Login值添加到枚举常量中， Notice::Type记录定义新的通知类型。然后，脚本将调用NOTICE和定义 $note，$msg，$sub，$id，和$uid记录。
其次，通知加入系统通过Notice::policy挂钩进行管理 。一个Notice::policy钩其参数Notice::Info用来存放脚本在其调用时提供的信息记录。通过访问Notice::Info中特定的记录，进行逻辑判断，以更改用于处理系统中的通知的策略。

### 通知类型
通知参数 | 描述
- | -
ACTION_NONE | 不采取行动
ACTION_LOG | 将通知发送到Notice :: LOG日志记录流。
ACTION_EMAIL | 发送带有通知的电子邮件。
ACTION_ALARM | 将通知发送到Notice :: Alarm_LOG流。然后每小时进行一次轮播，并将其内容以可读ASCII电子邮件发送到以下地址： Notice::mail_dest。


示例中，我们已添加Notice:: ACTION_LOG到 n$actions集合中。该类型本身是可枚举的， 
Notice::ACTION_LOG操作将通知写入 Notice::LOG日志记录流，在默认配置中，日志将写入notice.log文件，并且不采取进一步操作。
Notice::ACTION_EMAIL操作会将电子邮件发送到Notice::mail_dest 变量中定义的一个或多个地址，并将通知的详细信息作为电子邮件的正文。
Notice::ACTION_ALARM将通知发送到Notice::ALARM_LOG日志记录流，然后每小时进行一次轮播，并将其内容以可读ASCII电子邮件发送到Notice::mail_dest 

### 脚本使用
+ 单一脚本   

将脚本以load方式添加到配置
```
echo ‘@load base/**' >> /usr/local/zeek/share/zeek/site/local.zeek
```
重新加载
```
/usr/local/zeek/bin/zeekctl deploy
```

+ 多个脚本组成的目录

如果指定目录名而不是文件名，则Zeek将尝试在该目录中加载名为" __load __.zeek"的文件（该文件将包含其他“ @load”指令）。
将目录地址以load方式添加到配置
```
echo ‘@load base/**' >> /usr/local/zeek/share/zeek/site/local.zeek
```
重新加载
```
/usr/local/zeek/bin/zeekctl deploy
```
