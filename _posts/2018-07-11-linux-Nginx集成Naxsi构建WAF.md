---
layout: post
category: "linux"
title:  "Nginx集成Naxsi构建WAF"
tags: [linux, nginx,naxsi]
---

### Nginx集成Naxsi构建WAF

Naxsi是一个开放源代码、高效、低维护规则的Nginx Web应用防火墙模块，Naxsi的主要目标是帮助人们加固Web应用程序，以抵御SQL注入、跨站脚本、跨域伪造请求、本地和远程文件等包含的漏洞。  
官网：https://github.com/nbs-system/naxsi    

#### 安装Naxsi
下载   
下载nginx和naxsi，这里下载的版本如下：  
	nginx-1.12.2.tar.gz   
	naxsi-0.55.3.tar.gz   

安装依赖  
	yum install pcre pcre-devel  libxml2 libxml2-devel zlib zlib-devel openssl openssl-devel   

解压安装包，这里解压到/opt/apps下   
	# tar xzvf naxsi-0.55.3.tar.gz
	# tar xzvf nginx-1.9.2.tar.gz
	# cd nginx-1.9.2
	# ./configure --add-module=/opt/apps/naxsi-0.55.3/naxsi_src/
	# make && make install

#### 配置Naxsi
**核心规则**  
首先需要将Naxsi核心规则文件naxsi_core.rules引入到Nginx主配置文件中。  
	cp /opt/apps/naxsi-0.55.3/naxsi_config/naxsi_core.rules  /usr/local/nginx/conf/

修改Nginx主配置文件，在http区块中插入以下内容：  
\# vim /usr/local/nginx/conf/nginx.conf  

	http {
	...
	include /usr/local/nginx/conf/naxsi_core.rules;
	...
	}

**子规则**  
新建子规则  
\# vim  /usr/local/nginx/conf/naxsi.rules  

	# 启用Naxsi模块
	SecRulesEnabled;
	#启用学习模式，即拦截请求后不拒绝访问，只将触发规则的请求写入日志
	LearningMode;
	# 拒绝访问时展示的页面
	DeniedUrl "/RequestDenied";
	# 检查规则
	CheckRule "$SQL >= 8" BLOCK;
	CheckRule "$RFI >= 8" BLOCK;
	CheckRule "$TRAVERSAL >= 4" BLOCK;
	CheckRule "$EVADE >= 4" BLOCK;
	CheckRule "$XSS >= 8" BLOCK;
	error_log  /var/log/nginx/naxsi.log;

以上配置的作用是启用Naxsi模块并拦截指定的非法请求。如果要关闭Naxsi模块，可使用SecRulesDisabled选项。  
学习模式只是用来帮助你在不影响正常访问的情况下找到触发安全规则的合法请求，然后将触发规则写入naxsi.log。如果触发会增加如下日志信息：  
	2018/07/11 11:24:08 [error] 2465#0: *111 NAXSI_FMT: 				ip=10.72.79.129&server=10.110.8.34&uri=/&learning=0&vers=0.55.3&total_processed=1&total_blocked=1&block=1&cscore0=$SQL&score0=8&zone0=ARGS&id0=1000&var_name0=id, client: 10.72.79.129, server: localhost, request: "GET /?id=select%20*%20from HTTP/1.1", host: "10.110.8.34"

注意在最后生产环境一定要把学习模式关闭，不然是没有拒绝访问防御的效果。  
DeniedUrl是把拒绝的请求发送到内部URL，CheckRule是设置各规则不同的触发阈值。 一旦该阈值触发，请求将被阻塞。  

接下来，我们在Server区块中引入上面定义的Naxsi子规则。  

\# vim /usr/local/nginx/conf/nginx.conf  

	server {
		....
		location / {
		# 引用Naxsi子规则
		include  /usr/local/nginx/conf/naxsi.rules;
		root   html;
		index  index.html index.htm;
		}
		# 配置拦截后拒绝访问时展示的页面，这里直接返回403。
		location /RequestDenied {
			return 403;
		}
	}

验证配置文件   
	/usr/local/nginx/sbin/nginx -t  

启动、测试  
	/usr/local/nginx/sbin/nginx

正常访问页面：  
![正常访问]({{"/assets/linux/201807/1101.png" | absolute_url}})

访问地址添加?message=system(%27ls%20-l%27)，会转跳到拦截页面  
![脚本注入]({{"/assets/linux/201807/1102.png" | absolute_url}})

访问地址添加sql，?id=select%20*%20from   
![sql注入]({{"/assets/linux/201807/1103.png" | absolute_url}})

#### 配置Naxsi白名单
**白名单规则**   

wl:ID (White List ID) 哪些拦截规则会进入白名单。正确的语法是:  
	wl:0:把所有拦截规则加入白名单
	wl:42:把ID为42的拦截规则加入白名单
	wl:42,41,43:把ID为42, 41和43的拦截规则加入白名单
	wl:-42:把所有拦截规则加入白名单，除了ID为42的拦截规则

mz:(Match Zones) 指定的区域会生效本条白名单。  
	ARGS: GET的整个参数，如: foo=bar&in=%20
	$ARGS_VAR: GET参数的参数名, 如：foo=bar&in=%20中的foo和in
	$ARGS_VAR_X: 正则匹配的GET参数的参数名
	HEADERS: 整个HTTP协议头
	$HEADERS_VAR: HTTP协议头的名字
	$HEADERS_VAR_X: 正则匹配的HTTP协议头的名字
	BODY: POST的整个参数内容
	$BODY_VAR: POST参数的参数名
	$BODY_VAR_X: 正则匹配的POST参数的参数名
	URL: URL(?前的)
	$URL_X: 正则匹配的URL(?前的)
	FILE_EXT: 文件名 (POST上传文件时上传的文件名）

**设置白名单**    
\# vim /usr/local/nginx/conf/naxsi_whitelists.rules  
	BasicRule wl:1000 "mz:ARGS";

以上设置对于#1000号规则，将所有的GET请求参数加入白名单，#1000规则主要是sql关键字，配置此条信息之后，之前的sql注入测试将会通过拦截
