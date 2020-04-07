title: PHP和MySQL处理emoji表情
date: 2016-03-23 11:28:53
tags: [php,mysql]
categories: 学习笔记

---
### 写在前面
　　前面我写了一篇关于字符集的文章和一篇关于MySQL字符集和校对的文章,现在我再来想想如何解决一开遇到的那个emoji表情处理的问题。  
### php  
　　php处理emoji表情的方式主要有两种：
　　1. 完全过滤emoji表情,就是去掉字符串中的emoji表情,这样虽然对提高程序健壮性有帮助,但是缺损失了用户体验
<!-- more -->
　　2. 另外一个办法就是转义emoji表情字符,存入数据库,然后取出来时再转义回来。
　　在之前介绍utf-8编码时,有说到utf-8编码的字符串如果一个字节的第一位是0，则这个字节单独就是一个字符；如果第一位是1，则连续有多少个1，就表示当前字符占用多少个字节。因为emoji表情需要处理处理主要是因为MySQL数据库utf-8字符集只能存储3个字节长度的字符,而emoji表情是4个字符长度的,而5个甚至是6个字节长度的unicode字符肯定也是不能存储的,所以也需要转换。
　　在收到一段可能含有emoji表情的文本内容后，可以简单的使用 json_encode($str) 将其进行JSON编码，此时消息中的表情、中文等字符将会被转为unicode编码显示。（这里进行JSON编码就是为了获得字符的unicode码，所以json_encode函数中不需要增加避免unicode的可选参数了）,然后根据unicode码筛选出需要过滤或者转换的字符,存入数据库。如果是转换的换,在取出来之后还需要还原。  
　　这里有一个php转换emoji表情的库 github.com/iamcal/php-emoji ,这样的做法缺点就是每次新增了emoji表情之后还需要在库里面添加对应的键值对映射,就会很麻烦。
### MySQL  
　　因为要处理emoji表情的根本问题在于MySQL的存储问题,所以只要修改MySQL的字符集使之能够处理4个字节长度就可以了。  
　　因为MySQL在处理数据和连接的时候,可能会发生字符集转换的主要是三个地方：客户端连接,数据表存储,和结果返回。所以在这三个要设置正确的字符集,来存储emoji表情,MySQL的utf8mb4是utf8的超集,能存储下4个字节的数据。MySQL数据库版本要5.5.3及以上才支持utf8mb4。  
　　数据库设置中与字符集相关的一些设置：  
　　1. character_set_server：默认的内部操作字符
　　2. character_set_client：客户端来源数据使用的字符集
　　3. character_set_connection：连接层字符集
　　4. character_set_results：查询结果字符集
　　5. character_set_database：当前选中数据库的默认字符集
　　6. character_set_system：系统元数据(字段名等)字符集
　　7. 还有以collation_开头的同上面对应的变量，用来描述字符集。  
### 参考资料
　　[深入Mysql字符集设置](http://www.laruence.com/2008/01/05/12.html)