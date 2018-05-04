---
layout: post
category: "web"
title:  "sonar、jenkins构建代码检查"
tags: [web, 代码检查, sonar, jenkins]
---

### sonar、jenkins构建代码检查

最近因为工作原因构建了一下代码检查流程，这里记录一下

#### 安装sonar  
##### 预置条件  
	1）已安装JAVA环境
		版本：JDK1.8
	2）已安装有mysql数据库
		版本：mysql5.6以上
	3）下载sonarQube与sonar-scanner
		版本：sonarQube5.6
		版本：sonar-scanner2.8

##### 创建数据库  
创建用户sonar：  

·CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
	CREATE USER 'sonar' IDENTIFIED BY 'sonar';
	GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonar';
	GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';
	FLUSH PRIVILEGES;·

##### 修改配置文件
	将sonar-5.6.zip上传到服务器，放置到/home目录下，新建sonar目录，并解压到当前目录即可。
	修改conf目录下的sonar.properties文件
	配置参考：  

![数据库连接]({{"/assets/web/1.png" | absolute_url}})

	修改数据库连接及用户名、密码

![访问地址IP]({{"/assets/web/2.png" | absolute_url}})

	同时需要设置设置 sonar.scm.disabled=true
	web界面配置需修改如下，如果不设置，通过svn访问会报无权限  

![scm修改]({{"/assets/web/3.png" | absolute_url}})

##### 启动服务
	启动sonar
	切换到sonar安装目录下 /bin/linux-x86-64
	#./sonar.sh start

	访问http:\\IP:9000页面出现sonarQube页面即可（启动成功）

#### sonar插件安装
##### 中文插件安装
	1.进入插件下载页面
		http://docs.codehaus.org/display/SONAR/Plugin+Library
	2.找到Localization---chinese   点击，选择相应版本对应的插件下载
	3.下载后，放入sonar目录如下sonarqube-5.6\extensions\plugins，重启sonar，访问页面如下：

![访问页面]({{/assets/web/4.png" | absolute_url}})

##### 其余插件集成
	下载SonarWeb、SonarXML插件，放入sonar目录如下sonarqube-5.6\extensions\plugins，重启sonar

#### sonar-scanner部署
##### 安装、配置
	目录下解压文件，修改配置文件sonar-scanner.properties，配置文件内容包括数据库连接及访问地址

![配置文件]({{/assets/web/5.png" | absolute_url}})

	配置环境变量
	在/etc/profile下配置环境变量

![环境变量]({{/assets/web/6.png" | absolute_url}})

##### 代码扫描
	创建sonar-project.properties。
	对代码进行分析，必须要创建sonar-project.properties文件，执行sonar-scanner时，会对根据
	sonar-project.properties文件进行搜索，该文件要放在与项目同一目录下。

![扫描文件配置]({{/assets/web/7.png" | absolute_url}})

	执行sonar-scanner。进入到存放sonar-project.properties文件的目录，执行sonar-scanner，执行结果显示EXECUTION SUCCESS，则解析成功。
	此时登陆http://IP:9000，会看到解析project的信息在网页上显示

![扫描结果]({{/assets/web/8.png" | absolute_url}})

#### Jenkins部署及集成sonar
##### Jenkins部署
	下载jenkins.war，Tomcat下部署，启动Tomcat，访问http://ip:port/jenkins，管理员权限登录Jenkins，
	安装插件，两种方式:
	1、启动时选择安装插件，需要连接网络，内网环境受限
	2、选择：系统管理-->插件管理—>高级，上传插件，将下载的插件上传，之后重启jenkins。

	需要的插件为svn插件及sonar插件，如下图：

![插件]({{/assets/web/9.png" | absolute_url}})

	重启Jenkins后，管理员权限登录进入：系统管理--> 系统设置 
	配置如下：

![sonar配置]({{/assets/web/10.png" | absolute_url}})

	Server URL切记加上http://，不能只输入IP地址加端口，否则会提示不能连接，Server authentication token项需要在sonar server端生成，访问sonar server端，菜单--》配置--》权限--》用户--》TOKENS--》Generate ，输入TokenName=”admin”，生成token，将生成的复制到Server authentication token

	管理员权限登录进入：系统管理--> 全局工具配置
	设置JDK：

![JDK设置]({{/assets/web/11.png" | absolute_url}})

	设置SonarQube Scanner：

![SonarQube设置]({{/assets/web/12.png" | absolute_url}})

##### 新建任务
	填写项目名称：

![项目名]({{/assets/web/13.png" | absolute_url}})

	填写svn配置信息：

![svn信息]({{/assets/web/14.png" | absolute_url}})

	触发设置（根据需要设置）

![触发]({{/assets/web/15.png" | absolute_url}})

	构建信息（选择执行 sonarqube scanner）

![构建]({{/assets/web/16.png" | absolute_url}})

	之后返回会发现出现sonarqube选项，点击立即构建，构建完成之后，可以在sonarqube界面端查看检查结果
	
![执行构建]({{/assets/web/17.png" | absolute_url}})






