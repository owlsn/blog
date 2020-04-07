title: nginx编译安装与配置学习笔记
date: 2016-03-17 14:02:58
tags: [nginx]
categories: 学习笔记

---
### 写在前面  
　　这篇文章里面记录了我在自己独立安装配置nginx的时候遇到的很多问题,并且还记录了很多其他的与之或多或少有关联的知识。下面是安装的软件的下载地址:
　　[nginx-1.8.1](http://nginx.org/en/download.html)  
　　[openssl-1.0.1s](http://www.openssl.org/source/)  
　　[pcre](http://www.pcre.org/)  
### nginx编译安装  
　　首先在我自己新建的文件夹下面`/var/workspace/nginx`去nginx官网获取nginx压缩包  
　　`wget nginx.org/download/nginx-1.8.1.tar.gz`  
<!-- more -->
　　然后解压缩  
　　`tar zxvf nginx-1.8.1.tar.gz`  
　　进入解压后的目录,进行配置  
　　在这一步进行配置的时候有一些参数要注意下,  
　　`--prefix=`,`--sbin-path`以及一些其他的目录路径,这个根据自己情况设置就好了  
　　`--user=`和`--group=`这两个是设置nginx的运行用户和运行分组,在我看来就相当于是设置运行的权限的  
　　gzip模块需要 zlib 库,rewrite模块需要 pcre 库,ssl功能需要 openssl 库,所以有需要安装的功能就需要先安装一些其他的库,可以选择几个后面能用到的编译的时候就加载进来。如果现在没有加载也不要紧，另外也可以安装了nginx之后用`--add-module=`添加这些模块,在网上可以找到相关的资料。  
　　`configure`之后如果有报错的话,就根据相应的报错信息解决,一般都是缺少什么东西或者版本问题,把需要的东西装上就行了。
　　然后`make && make install`就行了。  
### nginx配置  
　　在nginx的安装目录下面,会有sbin和conf文件夹,conf文件夹里面存放的就是nginx的一些配置文件,首先就是nginx服务器本身的配置文件nginx.conf,里面的一些参数我在网上找了一些资料记录了一下
<pre>
	#运行用户
	user www-data;    
	#启动进程,通常设置成和cpu的数量相等
	worker_processes  1;

	#全局错误日志及PID文件
	error_log  /var/log/nginx/error.log;
	pid        /var/run/nginx.pid;

	#工作模式及连接数上限
	events {
    	use   epoll;             #epoll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能
    	worker_connections  1024;#单个后台worker process进程的最大并发链接数
    	#multi_accept on; 
	}

	#设定http服务器，利用它的反向代理功能提供负载均衡支持
	http {
    	#设定mime类型,类型由mime.type文件定义
    	include       /etc/nginx/mime.types;
    	default_type  application/octet-stream;
    	#设定日志格式
    	access_log    /var/log/nginx/access.log;

    	#sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
	    #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
	    sendfile        on;
	    #tcp_nopush     on;
	
	    #连接超时时间
	    #keepalive_timeout  0;
	    keepalive_timeout  65;
	    tcp_nodelay        on;
	    
	    #开启gzip压缩
	    gzip  on;
	    gzip_disable "MSIE [1-6]\.(?!.*SV1)";
	
	    #设定请求缓冲
	    client_header_buffer_size    1k;
	    large_client_header_buffers  4 4k;
	
	    include /etc/nginx/conf.d/*.conf;
	    include /etc/nginx/sites-enabled/*;
	
	    #设定负载均衡的服务器列表
	upstream mysvr {
		#weigth参数表示权值，权值越高被分配到的几率越大
	    #本机上的Squid开启3128端口
	    server 192.168.8.1:3128 weight=5;
	    server 192.168.8.2:80  weight=1;
	    server 192.168.8.3:80  weight=6;
	    }

	server {
	    #侦听80端口
	        listen       80;
	        #定义使用www.xx.com访问
	        server_name  www.xx.com;
	
	        #设定本虚拟主机的访问日志
	        access_log  logs/www.xx.com.access.log  main;
	
	    #默认请求
	    location / {
	          root   /root;      #定义服务器的默认网站根目录位置
	          index index.php index.html index.htm;   #定义首页索引	文件的名称
	
	          fastcgi_pass  www.xx.com;
	         fastcgi_param  SCRIPT_FILENAME  $document_root/	$fastcgi_script_name; 
	          include /etc/nginx/fastcgi_params;
	        }
	
	    # 定义错误提示页面
	    error_page   500 502 503 504 /50x.html;  
	        location = /50x.html {
	        root   /root;
	    }
	
	    #静态文件，nginx自己处理
	    location ~ ^/(images|javascript|js|css|flash|media|static)/ {
	        root /var/www/virtual/htdocs;
	        #过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更	新，则可以设置得小一点。
	        expires 30d;
	    }
	    #PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
	    location ~ \.php$ {
	        root /root;
	        fastcgi_pass 127.0.0.1:9000;
	        fastcgi_index index.php;
	        fastcgi_param SCRIPT_FILENAME /home/www/www$fastcgi_script_name;
	        include fastcgi_params;
	    }
	    #设定查看Nginx状态的地址
	    location /NginxStatus {
	        stub_status            on;
	        access_log              on;
	        auth_basic              "NginxStatus";
	        auth_basic_user_file  conf/htpasswd;
	    }
	    #禁止访问 .htxxx 文件
	    location ~ /\.ht {
	        deny all;
	    }
		}
	}
</pre>
　　以及负载均衡设置
<pre>
	#设定http服务器，利用它的反向代理功能提供负载均衡支持
	http {
	     #设定mime类型,类型由mime.type文件定义
	    include       /etc/nginx/mime.types;
	    default_type  application/octet-stream;
	    #设定日志格式
    	access_log    /var/log/nginx/access.log;

	    #省略上文有的一些配置节点

	    #。。。。。。。。。。

    	#设定负载均衡的服务器列表
    	 upstream mysvr {
    	#weigth参数表示权值，权值越高被分配到的几率越大
    	server 192.168.8.1x:3128 weight=5;#本机上的Squid开启3128端口
    	server 192.168.8.2x:80  weight=1;
    	server 192.168.8.3x:80  weight=6;
    	}

	   upstream mysvr2 {
    	#weigth参数表示权值，权值越高被分配到的几率越大
	
    	server 192.168.8.x:80  weight=1;
    	server 192.168.8.x:80  weight=6;
    	}
	
	   #第一个虚拟服务器
	   server {
    	#侦听192.168.8.x的80端口
    	    listen       80;
    	    server_name  192.168.8.x;

	      #对aspx后缀的进行负载均衡请求
    	location ~ .*\.aspx$ {

    	     root   /root;      #定义服务器的默认网站根目录位置
    	      index index.php index.html index.htm;   #定义首页索引文件的名称

    	      proxy_pass  http://mysvr ;#请求转向mysvr 定义的服务器列表

    	      #以下是一些反向代理的配置可删除.

    	      proxy_redirect off;

    	      #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
	          proxy_set_header Host $host;
    	      proxy_set_header X-Real-IP $remote_addr;
    	      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	          client_max_body_size 10m;    #允许客户端请求的最大单文件字节数
	          client_body_buffer_size 128k;  #缓冲区代理缓冲用户端请求的最大字节数，
	          proxy_connect_timeout 90;  #nginx跟后端服务器连接超时时间(代理连接超时)
	          proxy_send_timeout 90;        #后端服务器数据回传时间(代理发送超时)
	          proxy_read_timeout 90;         #连接成功后，后端服务器响应时间(代理接收超时)
	          proxy_buffer_size 4k;             #设置代理服务器（nginx）保存用户头信息的缓冲区大小
	          proxy_buffers 4 32k;               #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
	          proxy_busy_buffers_size 64k;    #高负荷下缓冲大小（proxy_buffers*2）
	          proxy_temp_file_write_size 64k;  #设定缓存文件夹大小，大于这个值，将从upstream服务器传
	       }
     }
	}
</pre>  
　　另外关于nginx和php的一些相关的安全配置,我在网上找到了一片讲解这方面比较详细的文章,附上链接[Nginx安全配置研究](http://drops.wooyun.org/tips/1323),[nginx缓存配置](https://linux.cn/article-5945-1.html),[nginx添加模块](http://coderschool.cn/1728.html)
### 参考资料
　　[Nginx安全配置研究](http://drops.wooyun.org/tips/1323)
　　《Nginx模块开发与架构解析》