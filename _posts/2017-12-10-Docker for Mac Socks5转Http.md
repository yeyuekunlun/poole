---
layout: post
title: Docker for mac 通过polipo或privoxy使用Shadowsocks
---

*这篇文章合适那些正在寻求如何在Mac终端使一些不支持sock5协议的软件可以免于被墙，加快境外链接的访问速度。以及如何在Mac终端下直连Docker Hub*

这两天一直被如何在Mac终端的docker上使用shadowsocks连接Docker Hub而困扰。经过一些折腾，问题总算是得以解决了，下面就把问题解决的过程记录一下。

-----

## Mac终端下使不支持Sock5协议的应用能使用Shadowsocks代理

解决办法是创建http代理，然后把请求转发给本地的ss。这可以选择使用polipo或者是privoxy

### I  使用privoxy实现http代理

##### 1.安装privoxy
	brew install privoxy
	
##### 2.查看安装信息
	brew info privoxy
	
>To have launchd start privoxy now and restart at login:
  brew services start privoxy
Or, if you don't want/need a background service you can just run:
  privoxy /usr/local/etc/privoxy/config
  
##### 3.修改配置文件

	listen-address  127.0.0.1:8118
	forward-socks5 / 127.0.0.1:1080 .
	
##### 4.用你喜爱的方式启动privoxy

	brew service start privoxy

##### 5.查看是否启动成功

查看是否启动成功

	ps aux|grep privoxy
	netstat -an|grep 8118
	或者
	lsof -i: 8118
	
测试
	
	curl ipinfo.io 
	export http_proxy="http://127.0.0.1: 8118"
	curl ipinfo.io
	
### II 使用polipo实现Http代理

##### 1.安装polipo

>Polipo  is  a  caching  HTTP  proxy.   It  listens to requests for web pages from your browser and forwards them to web servers, and forwards the servers'
       replies to your browser.  In the process, it optimises and cleans up the network traffic.
       
	brew install polipo
	
##### 2.修改配置文件

polipo的配置文件通过`man polipo`查看说明

>/etc/polipo/config The default location of Polipo's configuration file.

添加如下配置

	//The address and TCP port on which polipo wil listen for client requests.
	proxyAddress = "127.0.0.1"
	//polipo的监听端口
	proxyPort = 8123
	//shadowsocks监听地址和端口
	socksParentProxy = "127.0.0.1:1080"
	socksProxyType = socks5

##### 3.启动polipo

通过`polipo`命令直接启动即可，有可能需要sudo

##### 4.查看是否启动成功（同privoxy)

### III 如何配置Docker for Mac使用Http代理

##### 1.设置Docker的preference并重启Docker

![1](/my_img/1.png)

*特别注意这里的IP地址要填写你自己本机的真实ip,因为Mac版本的Docker命令是运行在虚拟机中的*

![2](/my_img/2.png)

##### 2.修改I,II中配置的监听地址
改为0.0.0.0表示接受来自网络中的其它主机的连接请求,并重启之

	listen-address  0.0.0.0:8118
	或
	proxyAddress = "0.0.0.0"
	
##### 3.最后测试一下吧，相信可以愉快的玩耍啦
