# 第1章 Nginx简介

### 常见web服务器

- Apache

  世界上用的最多的Web服务器。源码开放。重量级产品，消耗内存高。速度、性能上不及其他轻量级Web服务器。

- Lighttpd

  轻量级Web服务器，Nginx的竞争对手

- Tomcat

  运行servlet和JSP Web应用软件的基于Java的Web应用软件容器。

  Tomcat对静态文件、高并发的处理比较弱

### Nginx的发展

Nginx：高性能的HTTP和反向代理服务器

Nginx能够选择高效的epull、kqueue、eventport作为网络的I/O模型

它能够支持高达50000个并发连接数的响应，而内存、CPU等资源消耗却非常低

- 它可以高并发连接
- 内存消耗少
- 成本低廉

其他：

- 配置文件非常简单

- 支持Rewrite重写规则

  能够根据域名、URL的不同，将HTTP请求分发到不同的后段服务器群组。

- 内置的健康检查

- 节省带宽

  支持GZIP压缩，可以添加浏览器本地缓存的Header头

- 稳定性高

- 支持热部署

![](http://p1yseh5av.bkt.clouddn.com/18-8-2/33611479.jpg)



# 第2章 Nginx服务器的安装与配置

- Mac下安装

  `brew install nginx`

- Nginx启动

  `nginx -c nginx.conf`

- Nginx的停止

  查找nginx主进程号：ps -ef | grep nginx

  `nginx -s stop`

  - 从容停止nginx

    `kill -QUIT Nginx主进程号`

  - 快速停止nginx

    `kill -TERM Nginx主进程号`

    或者 `kill -INT Nginx主进程号`

  - 强制停止所有Nginx进程

    `pkill -9 nginx`

- Nginx的平滑重启

  `nginx -s reload`

   判断配置文件是否正确：`nginx -t -c nginx.conf`

   然后 `kill -HUP Nginx主进程号`



## 第3章 Nginx的基本配置与优化

### Nginx的完整配置示例

nginx.conf的配置文件结构主要由以下几部分构成

```nginx
...
events
{
   ... 
}
http
{
	...
    server
    {
    	...    
    }
    
    server
    {
    	...    
    }
    
    ...
}
```

### Nginx的虚拟主机配置

#### 什么是虚拟主机

虚拟主机使用的是特殊的软硬件技术，**它把一台运行在因特网上的服务器主机分成一台台“虚拟”的主机**，每台虚拟主机都可以是一个独立的网站，可以具有独立的域名，具有完整的Internet服务器功能（WWW、FTP、Email等），同一台主机上的虚拟主机之间是完全独立的，从网站访问者来看，每一台虚拟主机和一台独立的主机完全一样。

利用虚拟主机，不用为每个要运行的网站提供一台单独的Nginx服务器或单独运行一组Nginx进程。虚拟主机提供了在同一台服务器、同一组Nginx进程上运行多个网站的功能。

一个最简化的虚拟主机配置代码如下所示：

 ```nginx
http
{
    server
    {
    	listen 80 default;
        server_name _*;
        access_log  logs/default.access.log combined;
        location / {
            index index.html;
            root /data0/htdocs/htdocs;
        }
    }
}
 ```

跟Apache一样，Nginx也可以配置多种类型的虚拟主机：

- 基于IP的虚拟主机
- 基于域名的虚拟主机
- 基于端口的虚拟主机

#### 配置基于IP的虚拟主机

Linux、FreeBSD操作系统都允许添加IP别名

```nginx
http
{
    # 第一个虚拟主机
    server
    {
        # 监听的IP和端口
    	listen    192.168.8.43:80;
        # 主机名称
        server_name    192.168.8.43;
        access_log  logs/server1.access.log combined;
        location / {
            # 默认首页文件，从左到右
            index index.html index.htm
            #html 网页存放的根目录    
            root /data0/htdocs/server1;    
        }
    }
    server
    {
    	listen    192.168.8.44:80;
        server_name    192.168.8.43;
        access_log  logs/server2.access.log combined;
        location / {
            index index.html index.htm 
            root /data0/htdocs/server2;    
        }
    }
}
```

#### 配置基于域名的虚拟主机

基于域名的虚拟主机是最常见的一种虚拟主机。只需要配置你的DNS服务器，将每个主机名映射到正确的IP地址，然后配置Nginx服务器，令其识别不同的主机名就可以了。这种虚拟主机技术，使很多虚拟主机可以共享同一个IP地址，有效解决了IP地址不足的问题。

```nginx
http
{
    # 第一个虚拟主机
    server
    {
        # 监听的IP和端口
    	listen    80;
        # 主机名称
        server_name   a.domain.com;
        access_log  logs/server1.access.log combined;
        location / {
            # 默认首页文件，从左到右
            index index.html index.htm
            #html 网页存放的根目录    
            root /data0/htdocs/server1;    
        }
    }
    server
    {
    	listen    80;
        server_name    b.domain.com;
        access_log  logs/server2.access.log combined;
        location / {
            index index.html index.htm 
            root /data0/htdocs/server2;    
        }
    }
}
```





