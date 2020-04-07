title: javaweb环境搭建
date: 2018-10-05 10:32:00
tags: [java]
categories: 环境搭建

---
### 写在前面  
　　 最近开始学习java和大数据相关的知识，希望以后能从事这方面的工作，而且很长一段时间没有写代码了，相关的一些知识再记录熟悉一下
### 正文  
　　系统环境：ubuntu16.04.   
    1.安装java.   
	`sudo apt-get install java-8-oracle`.   
	配置环境变量。  
	`vim /etc/profile`
	`export JAVA_HOME=/usr/lib/jvm/java-8-oracle`. 
	`export JRE_HOME=${JAVA_HOME}/jre`
	`export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  `
	`export PATH=${JAVA_HOME}/bin:$PATH`
	`source /etc/profile`.   
    2.安装tomcat8.   
	`sudo apt-get install tomcat8`。  
	配置稍后再说
	 3.安装mysql.   
	 `sudo apt-get install mysql-server`.   
	 `#配置文件在/etc/mysql/下`。  
	 `#修改bind-address开启远程访问`.   
	 `#grant all privileges on *.* to 'root'@'%' identified by 'password' with grant option`.   
	 `flush privileges`。  
	 4.安装maven。  
	 classpath  
	 5.安装hadoop集群。  
	 `#下载hadoop`   
	 `修改hadoop-env.sh,core-site.xml,hdfs-site.xml,mapred-site.xml,yarn-site.xml配置文件，集群还要添加workers配置，添加hosts和环境变量`
### 参考资料  
