title: cookie和session学习笔记
date: 2016-03-14 11:16:33
tags: [cookie,session]
categories: 学习笔记  

---
### 写在前面  
　　HTTP协议最初是一个匿名、无状态的请求/ 响应协议。服务器处理来自客户端的请求，然后向客户端回送一条响应。Web 服务器几乎没有什么信息可以用来判定是哪个用户发送的请求，也无法记录来访用户的请求序列  
### http首部
　　在现在的http首部中，有七种最常见的用来承载用户相关信息的HTTP 请求首部  

首部名称 | 首部类型 | 描　　述
---- | ---- | -----
From | 请求 | 用户的E-mail 地址
User-Agent | 请求 | 用户的浏览器软件
Referer | 请求 | 用户是从这个页面上依照链接跳转过来的
Authorization | 请求 | 用户名和密码
Client-IP | 扩展（请求）| 客户端的IP 地址
X-Forwarded-For | 扩展（请求）| 客户端的IP 地址
Cookie | 扩展（请求） | 服务器产生的ID 标签  
<!-- more -->
　　就以访问本博客请求头为例:  
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/request_header.png)  
　　由于担心一些服务器会收集用户的email,所以很少有浏览器会发送From首部,一般情况写,web机器人或者爬虫可能会带有From首部。  
　　User-Agent 首部可以将用户所用浏览器的相关信息告知服务器，包括程序的名称和版本，通常还包含操作系统的相关信息。要实现定制内容与特定的浏览器及其属性间的良好互操作时，这个首部是非常有用的，但它并没有为识别特定的用户提供太多有意义的帮助。在上图中就带有谷歌浏览器的一些版本信息。  
　　Referer 首部提供了用户来源页面的URL。Referer 首部自身并不能完全标识用户,但它确实说明了用户之前访问过哪个页面。通过它可以更好地理解用户的浏览行为，以及用户的兴趣所在  
　　From,User-Agent,Referer首部尽管都能得到一些有关用户的信息,但是却不能实现可靠的识别。  
　　web服务器无需被动地猜测用户的身份,它可以要求用户通过用户名和密码进行认证（登录）来显式地询问用户是谁。如果服务器希望在为用户提供对站点的访问之前，先行登录，可以向浏览器回送一条HTTP 响应代码401 Login Required。然后，浏览器会显示一个登录对话框，并用Authorization 首部在下一条对服务器的请求中提供这些信息。在之后对这个站点的浏览器请求中,如果服务器需要登录验证,浏览器就会将存储的Authorization值发送出去,这样做虽然可以验证用户的身份,但是这样做也有很大的缺点:从一个站点浏览到另一个站点的时候，需要在每个站点上登录。  
　　Client-IP可以用来存储客户端的ip地址，用以区分不用的客户端。但是在Clinet-IP中存的有可能不是客户端的ip的地址。这里面可能是共享的NAT防火墙ip地址,或者是http代理的ip地址。  
　　X-Forwarded-For:简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项。  
### cookie  
　　可以笼统地将cookie分为两类：会话cookie 和持久cookie。会话cookie 和持久cookie 之间唯一的区别就是它们的过期时间。  
　　如果设置了Discard 参数，或者没有设置Expires 或Max-Age 参数来说明扩展的过期时间，这个cookie 就是一个会话cookie。  
#### cookie的工作过程  
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/cookie_process.png)  
　　cookie 的基本思想就是让浏览器积累一组服务器特有的信息，每次访问服务器时都将这些信息提供给它,将来用户返回同一站点时,浏览器会挑中那个服务器贴到用户上的那些cookie，并在一个cookie 请求首部中将其传回。
#### cookie的属性  
　　cookie中的主要内容包括一些属性值,有name,value,version,discard(当客户端程序终止时要丢弃此cookie,有此属性的一般都是会话cookie),domain,max-age(过期时间),path,port,secure(如果包含此属性,则只有在安全的ssl连接下才能发送cookie)等  
　　php中的setcookie()函数中的一些参数就对应了cookie中的一些属性
#### cookie的域属性和路径属性  
　　浏览器内部的cookie 罐中可以有成百上千个cookie，但浏览器不会将每个cookie都发送给所有的站点。产生cookie 的服务器可以向Cookie添加一个Domain 属性来控制哪些站点可以看到那个cookie。浏览器会根据这个来选择对应的cookie来发送给服务端。  
　　cookie 规范也允许用户将cookie 与部分Web 站点关联起来。可以通过Path 属性来实现这一功能，在这个Path属性列出的URL 路径前缀下所有cookie 都是有效的  
　　谷歌浏览器下查看本博客网站cookie例子  
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/cookie_preview.png)  
### session  
　　 session通常用于存储需要在整个用户会话过程中保持其状态的信息。  
#### session的工作原理  
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/session_process.png)  
　　1. 用户第一次访问服务器时,服务器会为用户创建sessionID,sessionID是对用户的唯一标识。  
　　2. 然后服务器会用session_register函数存储和用户状态有关的一些信息。  
　　3. 当脚本执行完成生成响应时,未被销毁的session变量会存储在对应的session.save_path中,然后sessionID会存储在响应头里的cookie中返回给客户端  
　　4. 用户第二次访问时,会带着本地的cookie内容。服务器会检测其中是否带有sessionID,如果没有的话就会再创建一个;如果有的话就根据sessionID的内容读取用户的状态信息  
　　5. 用户退出的话就是销毁对应的session变量和cookie。  
#### php session_start()  
　　1. session_start() 会创建新会话或者重用现有会话。 如果通过 GET 或者 POST 方式，或者使用 cookie 提交了会话 ID， 则会重用现有会话。  
　　2. 当会话自动开始或者通过 session_start() 手动开始的时候， PHP 内部会调用会话管理器的 open 和 read 回调函数。 会话管理器可能是 PHP 默认的， 也可能是扩展提供的（SQLite 或者 Memcached 扩展）， 也可能是通过 session_set_save_handler() 设定的用户自定义会话管理器。 通过 read 回调函数返回的现有会话数据（使用特殊的序列化格式存储）， PHP 会自动反序列化数据并且填充 $_SESSION 超级全局变量。
　　3. 要想使用命名会话，请在调用 session_start() 函数 之前调用 session_name() 函数。
　　4. session_start()之前不能有任何输出是因为在session_start()时,服务器会生成cookie首部,而如果有输出的话,内容就会在首部传回之前,造成错误。
#### session文件存储问题  
　　1. session性能问题。  
　　影响性能的原因之一由文件系统设计造成，在用户量很大的网站服务器上,在session存储目录下面肯定会存储非常多的session文件时，文件的定位将非常耗时。  
　　还有一个问题就是小文件的效率问题，一般我们的 session数据都不会太大（1～2K），如果有大量这样1～2K的文件在磁盘上，IO效率肯定会很差。  
　　其实还有很多中 存储session的方式，比如如果服务器装了memcached，还有会memcache的选项。另外还有很多，比如MySQL、PostgreSQL等等。  
　　2. session的同步问题。  
　　现在的网站大多都不止一台服务器了，当用户在一台服务器上登录之后,留下了自己的session信息,但是当用户访问另外一些页面却跳到了另外一台服务器上时,这台服务器上没有用户的session信息,这样就会产生问题。而对于这个,用memcache或者redis,MySQL存储的session信息,另外还有很多的解决方案,我自己也正在学习之中。  
### 参考资料  
　　《http权威指南》  
　　[session_start](http://php.net/manual/zh/function.session-start.php)