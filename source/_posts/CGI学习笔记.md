title: CGI学习笔记
date: 2016-02-26 22:04:31
tags: [cgi]
categories: 学习笔记

---
### 写在前面  
　　一开始接触php的时候是用的amp的开发环境,后来才知道公司线上的运行环境是lnmp的,既然要学习进阶,了解nginx的一些知识肯定还是非常有必要的。
### cgi  
　　在《http权威指南》中有提到  
<blockquote>
但最常见的网关，应用程序服务器，会将目标服务器与网关结合在一个服务器中实现。应用程序服务器是服务器端网关，与客户端通过 HTTP 进行通信，并与服务器端的应用程序相连
</blockquote>
<blockquote>
客户端是通过 HTTP 连接到应用程序服务器的。但应用程序服务器并没有回送文件，而是将请求通过一个网关应用编程接口（Application Programming Interface，API）发送给运行在服务器上的应用程序。
</blockquote>
<!-- more -->
<blockquote>
第一个流行的应用程序网关 API 就是通用网关接口（Common Gateway Interface，CGI）。CGI 是一个标准接口集，Web 服务器可以用它来装载程序以响应对特定 URL 的 HTTP 请求，并收集程序的输出数据，将其放在 HTTP 响应中回送。
</blockquote>
　　cgi的运行机制就如下图所示  
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/cgi.png)  
　　它在服务器和众多的资源类型之间提供了一种简单的、函数形式的粘合方式，用来处理各种需要的转换。这个接口还能很好地保护服务器，防止一些糟糕的扩展对它造成的破坏（如果这些扩展直接与服务器相连，造成的错误可能会引发服务器崩溃）。但是，这种分离会造成性能的耗费。为每条 CGI 请求引发一个新进程的开销是很高的，会限制那些使用CGI的服务器的性能，并且会加重服务端机器资源的负担，为了解决这个性能问题,人们开发了一种新型的cgi,就是fastcgi。
### fastcgi  
　　fastcgi模拟了cgi的功能,但是它是作为持久守护进程运行的,消除了为每个请求建立或拆除新进程所带来的性能损耗,它的工作原理：  
<blockquote>
1、Web Server启动时载入FastCGI进程管理器<br>
2、FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。<br>
3、当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。<br>
4、FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。<br>
<br>
在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。
</blockquote>
　　尽管cgi提供了一种简洁的接口方式,但是并不能改变服务器的行为,另外服务器还提供了一些额外的API,以便开发者将自己的模块与 HTTP 服务器直接相连。扩展 API 允许程序员将自己的代码嫁接到服务器上，或者用自己的代码将服务器的一个组件完整地替换出来。
　　整个服务程序应用网关机制可以用下图来描述
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/fastcgi.png)  
### php-fpm
　　PHP的解释器是php-cgi，它只是个CGI程序，只能解析请求，返回结果，不会进程管理。所以就出现了一些能够调度 php-cgi 进程的程序，比如说由lighthttpd 分离出来的spawn-fcgi。PHP-FPM也是这么个东西，在长时间的发展后，逐渐得到了大家的认可，也越来越流行。  