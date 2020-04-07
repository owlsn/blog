title: MySQL字符集与校对学习笔记
date: 2016-03-22 17:04:03
tags: [mysql]
categories: 学习笔记

---
### 写在前面
　　字符集指从二进制编码到字符符号的映射,而MySQL的校对指用于某个字符集的排序规则。MySQL4.1和之后的版本,每一类编码字符都有其对应的字符集和校对规则。
### MySQL如何使用字符集  
　　MySQL中有很多设置选项用于控制字符集,主要可以分为两类:创建对象时的默认值,在服务器和客户端通信时的设置。
#### 创建对象时的默认设置  
　　MySQL服务器,数据库,每个表都有自己的默认值,而且这个值是逐层继承的默认设置。这些是针对没有显式设置字符集的情况下才会使用的默认值。
<!-- more -->
#### 服务器和客户端通信时的设置  
　　服务器和客户端通信的时候可能会使用不同的字符集,这时服务端将进行必要的翻译转换工作:  
　　1. 服务器端总是假设客户端是按照character_set_client设置的字符集来传输数据和SQL的
　　2. 服务器收到数据和SQL时,先将其转换成字符集character_set_connection。
　　3. 当服务器返回数据或者错误信息时候,就会将其转换成字符集character_set_result。  
　　大致的过程可以用下图来表示
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/mysql_character.png)
#### 乱码产生原因
　　1. 向默认字符集为utf8的数据表插入utf8编码的数据前没有设置连接字符集，查询时设置连接字符集为utf8
　　2. 插入时根据MySQL服务器的默认设置，character_set_client、character_set_connection和character_set_results均为latin1；
　　3. 插入操作的数据将经过latin1=>latin1=>utf8的字符集转换过程，这一过程中每个插入的汉字都会从原始的3个字节变成6个字节保存；
　　4. 查询时的结果将经过utf8=>utf8的字符集转换过程，将保存的6个字节原封不动返回，产生乱码
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/utf-latin1-utf.png)
　　1. 向默认字符集为latin1的数据表插入utf8编码的数据前设置了连接字符集为utf8
　　2. 插入时根据连接字符集设置，character_set_client、character_set_connection和character_set_results均为utf8；
　　3. 插入数据将经过utf8=>utf8=>latin1的字符集转换，若原始数据中含有\u0000~\u00ff范围以外的Unicode字 符，会因为无法在latin1字符集中表示而被转换为“?”(0×3F)符号，以后查询时不管连接字符集设置如何都无法恢复其内容了。
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/utf-latin1.png)  
　　另外对于emoji字符,utf8字符集是用3个字节来保存的,而emoji字符占用了4个字节,所以当存入emoji表情时会出现报错的信息。
### 选择字符集和校对规则
　　对于字符集,合理的做法是首先为数据库设置好合适的字符集,然后在实际情况中为不同的列设置合适的字符集。  
　　对于校对规则,常常要考虑的问题就是是否要以大小写敏感的方式比较字符串,或者以字符串二进制编码的方式比较。他们对应的校对规则后缀分别为_cs(区分大小写),_ci(不区分大小写)和_bin(二进制比较)。
### 参考资料
　　[深入Mysql字符集设置](http://www.laruence.com/2008/01/05/12.html)    
　　《高性能MySQL》