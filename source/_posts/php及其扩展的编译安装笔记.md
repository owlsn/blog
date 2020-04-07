title: php及其扩展的编译安装笔记
date: 2016-03-18 22:04:16
tags: [php,memcached]
categories: 学习笔记

---
### 写在前面
　　这篇文章里面记录了我在自己独立安装开发环境的时候遇到的很多问题,并且还记录了很多其他的与之或多或少有关联的知识。
### 安装环境和安装软件  
　　我曾经来在虚拟机上安装过这些软件,在虚拟机上是在centos 6.7版本,而后我自己装的双系统上是centos 7,我觉得在实体机上操作一遍还是非常有必要的。  
　　这里我先把自己安装的这些软件的下载地址列出来：  
　　[memcached-1.4.25](http://memcached.org/downloads)  
　　[php-5.6.19](http://php.net/get/php-5.6.19.tar.gz/from/a/mirror)  
　　[libmemcached-1.0.18](https://launchpad.net/libmemcached/+download)  
　　[php-memcached扩展 2.2.0](https://pecl.php.net/package/memcached)  
　　这些需要安装的软件我将它们统一放在/var/workspace/下面对应的文件夹里面
<!-- more -->
### php安装  
　　安装之前首先看一看本机上面是不是已经装了其他版本的php,  
　　`yum -qa|grep php`  
　　我的电脑上之前yum安装过5.4版本的php,所以要先把原来的卸载掉  
　　`yum remove php`  
　　然后是获取php源码压缩包  
　　`wget hk1.php.net/get/php-5.6.19.tar.gz/from/this/mirror`  
　　下载完之后就解压文件得到`php-5.6.19`的文件夹  
　　然后`cd`进入`php-5.6.19`文件夹进行编译配置,配置之前我要先大概看看有哪些参数,这些参数一般用处是什么,网上很多资料在编译的时候后面都带了一大堆的参数,大部分都是在编译的时候就把扩展加载在里面了,甚至有很多都是默认的设置,根本不用加那些参数。但是实际上这些动态扩展可以之后再编译,然后再在php.ini添加就行了  
	<br>
　　`./configure --help|less`  
　　这里面主要有几个参数：  
　　`--prefix=`安装路径,  
　　`--with-config-file-path=`php.ini文件存放的路径,  
　　`--enable-fpm`开启php-fpm  
　　`--enable-mysqlnd`开启mysqlnd  
	<br>
　　安装路径和php.ini文件存放的路径我就不多说了,主要是下面两个,因为php-fpm和mysqlnd这两个都是已经整合进php源码里面了的,所以如果不重新编译php是不能以扩展的形式再加载到php里面的,但是这两个又是十分重要的。  
　　nginx本身是不能处理php脚本的,它只是将请求交给了php-cgi去处理,而php-fpm是管理php-cgi的管理器。因为是搭配nginx服务器,所以php-fpm是必须要装的。  
	<br>
　　对于mysqlnd(MySQL Native Driver),php手册中有写到
<blockquote>
MySQL Native Driver is a replacement for the MySQL Client Library (libmysqlclient). MySQL Native Driver is part of the official PHP sources as of PHP 5.3.0.
<br>
The MySQL database extensions MySQL extension, mysqli and PDO MYSQL all communicate with the MySQL server. In the past, this was done by the extension using the services provided by the MySQL Client Library. The extensions were compiled against the MySQL Client Library in order to use its client-server protocol.
<br>
With MySQL Native Driver there is now an alternative, as the MySQL database extensions can be compiled to use MySQL Native Driver instead of the MySQL Client Library.
</blockquote>
　　php5没有默认支持mysql了,如果要动态编译mysql或者mysqli扩展,都是要设置`/path/to/mysql_config`,这代表了要用来编译mysql客户端/连接mysql服务端的mysql_config程序位置,而php5.3之后引入了mysqlnd之后,就可以用mysqlnd选项代替mysql连接服务端的配置。  
　　而对于我现在编译的php5.6版本,如果不在编译的时候带参数`--enable-mysqlnd`直接支持mysqlnd,那么之后如果要编译mysql或者mysqli如果不显式设置mysql_config路径的话都是会出错的,因为5.4之后默认设置的就是mysqlnd,而编译文件中没有mysqlnd文件,所以就会报错。　　
	<br>
　　`./configure   
　　--prefix=/usr/local/php   
　　--with-enable-fpm   
　　--with-config-file-path=/usr/local/php/etc   
　　--with-enable-mysqlnd`  
　　然后就是配置和环境检查了,查看提示信息,缺少哪些依赖包,就到网上查找相关的资料,安装依赖包,最后等到没有错误之后就可以安装了。  
　　`make && make install`  
　　如果正常的话php就安装完成了,当然也有可能其中出现错误,对照错误信息去找解决办法解决就行了。安装完成了之后还没有完,因为设置的时候`/usr/local/php/etc`是php查找php.ini文件的地方,还要在php解压目录里复制一份php.ini-development文件过去。  
	<br>
　　php-fpm安装了之后还需要开启才能工作,在`/usr/local/php/sbin`文件夹下,有php-fpm命令,  
　　`/usr/local/php/sbin/php-fpm`  
　　启动php-fpm,另外的一些php-fpm的命令就可以自己查看帮助就行了。  
　　如果直接这样启动的话就可能出现`failed to open configuration file`,这个一看就知道是没有`php-fpm.conf`配置文件,到`/usr/local/php/etc/`目录下,从`php-fpm.conf.default`copy一份`php-fpm.conf`就行了。  
　　另外在php的解压目录中就有php-fpm的开机自启动脚本,复制一份到`/etc/init.d/`就可以了。    
### php扩展  
　　在解压出来的php文件夹中,就有一些扩展包文件,在ext文件夹下,如果在自己运行的程序中有需要某些扩展,一般都能在这里找到,我这里就以mysqli扩展为例。首先要cd进mysqli所在的目录  
　　`cd /var/workspace/php/php-5.6.19/ext/msyqli`  
　　php在安装完成之后会在安装目录的bin/文件夹下面生成一些命令,像phpize,php-config等,这些命令在编译扩展的时候就有用到。  
　　phpize在php官方文档中说是用来准备 PHP 扩展库的编译环境的,而php-config命令则是一个简单的命令行脚本用于获取所安装的 PHP 配置的信息。  
	<br>
　　执行phpize命令  
　　`/usr/local/php/bin/phpize`  
　　这里因为没有把命令加入到环境变量中,所以就只能直接输路径执行了。  
　　这里可能会缺少autoconf,直接yum安装就行了,或者想自己编译安装当然也可以了。
　　然后就会生成一些配置文件，其中就有./configure(有时候生成的是./config好像),然后就是编译安装之类的了,其中有一点配置的时候就是要注意加上  
　　`--with-php-config=`php-config命令的路径  
　　php-config命令的路径就是上面所说的路径,只有这样编译之后生成的so文件才会在正确的目录下面。一般就是在php安装目录下的`lib/php/extensions/...`什么的文件夹下面。然后在php.ini文件里加上`extension=`so文件的具体路径就行了。  
	<br>
　　然后如果本身ext文件夹下面没有扩展包的话就需要自己下载扩展,然后还是和上面一样的执行步骤。如果有遇到错误的话根据错误信息解决问题就行了。
### memcached
　　编译memcached也是类似的操作,下载之后解压文件`./configure --prefix=`配置一些安装路径或者其他的依赖包的路径,如果有缺少什么依赖包,在这一步都一般会有相应的提示信息的,只要逐个安装之后并把路径填写到对应的参数里面就行了。  
	<br>
　　这里首先安装的只是memcached的服务端,就是启动memcached服务的服务器,如果要在php中使用memcached的话就还要安装memcached的扩展。php-memcached的扩展又依赖于libmemcached,在开头我已经有把这些软件的下载地址列出来了,所以一个一个逐个安装就行了。最后记得还有修改php.ini文件。
　　