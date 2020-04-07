title: linux用户分组以及文件和目录的权限学习笔记
date: 2016-03-16 15:37:50
tags: [linux]
categories: 学习笔记

---
### 写在前面  
　　linux分组权限管理在实际工作中会经常遇到,这篇文章我主要是对一些命令和概念进行了总结罗列,方便记忆。  
### 用户和组相关的配置文件    
　　1. /etc/passwd  
　　系统用户配置文件，是用户管理中最重要的一个文件。这个文件记录了Linux系统中每个用户的一些基本属性，并且对所有用户可读。  
　　/etc/passwd中每一行记录对应一个用户，每行记录又被冒号分割，其格式和具体含义如下：  
　　用户名:口令:用户标识号:组标识号:注释性描述:主目录:默认shell  
　　例如:`root:x:0:0:root:/root:/bin/bash`  
　　　　- 用户名：是代表用户账号的字符串
　　　　- 口令：存放着加密后的用户口令
　　　　- 用户标识号：就是用户的UID
　　　　- 组标识号：就是组的GID
　　　　- 注释性描述：字段是对用户的描述信息
　　　　- 主目录：也就是用户登录到系统之后默认所处的目录
　　　　- 默认shell：就是用户登录系统后默认使用的命令解释器
　　2. /etc/shadow文件  
　　用户密码文件，由于/etc/passwd文件是所有用户都可读的，这样就导致了用户的密码容易出现泄露，因此，linux将用户的密码信息从/etc/passwd中分离出来，单独的放到了一个文件中，这个文件就是/etc/shadow，该文件只有root用户拥有读权限，从而保证了用户密码的安全性。  
　　/etc/shadow文件内容的格式：  
　　用户名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:保留字段  
　　例如:`root:$1$S4xQtnCs$6KteE0/CWlqWrq5.5i9Rc/:16875:0:99999:7:::`  
　　　　- 用户名：是代表用户账号的字符串
　　　　- 加密口令：存放的是加密后的用户口令字串，如果此字段是“*”、“!”、“x”等字符，则对应的用户不能登录系统
　　　　- 最后一次修改时间：表示从某个时间起，到用户最近一次修改口令的间隔天数
　　　　- 最小时间间隔：表示两次修改密码之间的最小时间间隔
　　　　- 最大时间间隔：表示两次修改密码之间的最大时间间隔，这个设置能增强管理员管理用户的时效性
　　　　- 警告时间：表示从系统开始警告用户到密码正式失效之间的天数
　　　　- 不活动时间：此字段表示用户口令作废多少天后，系统会禁用此用户，也就是说系统不再让此用户登录，也不会提示用户过期，是完全禁用
　　　　- 失效时间：表示该用户的帐号生存期，超过这个设定时间，帐号失效，用户就无法登录系统了。如果这个字段的值为空，帐号永久可用
　　　　- 保留字段：linux的保留字段，目前为空，以备linux日后发展之用
　　3. /etc/group文件  
　　用户组配置文件，用户组的所有信息都存放在此文件中。  
　　/etc/group文件内容的格式：  
　　组名:口令:组标识号:组内用户列表  
　　例如:`bin:x:1:bin,daemon`  
　　　　- 组名：是用户组的名称，由字母或数字构成
　　　　- 口令：存放的是用户组加密后的口令字串，密码默认设置在/etc/gshadow文件中，而在这里用“x”代替，linux系统下默认的用户组都没有口令，可以通过gpasswd来给用户组添加密码
　　　　- 组标识号：就是组的GID
　　　　- 组内用户列表： 显示属于这个组的所有用户，多个用户之间用逗号分隔
　　4. /etc/login.defs文件  
　　用来定义创建一个用户时的默认设置，比如指定用户的UID和GID的范围，用户的过期时间、是否需要创建用户主目录等等  
　　5. /etc/default/useradd文件  
　　通过useradd命令不加任何参数创建一个用户是的默认配置
　　6. /etc/skel目录  
　　新建立的用户在主目录下默认拥有自己指定的配置文件
### 用户和组管理  
#### 添加、修改和删除用户命令useradd/usermod/userdel  
　　1. useradd  
　　useradd不加任何参数创建用户时，系统首先读取添加用户配置文件/etc/login.defs和/etc/default/useradd，根据这两个配置文件中定义的规则添加用户，然后会向/etc/passwd和/etc/group文件添加用户和用户组记录，同时/etc/passwd和/etc/group对应的加密文件也会自动生成记录，接着系统会自动在/etc/default/useradd文件设定的目录下建立用户主目录，最后复制/etc/skel目录中的所有文件到新用户的主目录中，这样一个新的用户就建立完成了  
　　　　- 使用语法`useradd [-option] name`
　　　　- 常用参数
　　　　　　- -u uid：即用户标识号，此标识号必须唯一。
　　　　　　- -U user-group:创建一个与用户同名的分组,并将此用户加入分组。
　　　　　　- -g group：指定新建用户登录时所属的默认组，或者叫主组。此群组必须已经存在。
　　　　　　- -G group：指定新建用户的附加组，此群组必须已经存在。附加组是相对与主组而言的，当一个用户同时是多个组中的成员时，登录时的默认组成为主组，而其它组称为附加组。
　　　　　　- -c comment：对新建用户的说明信息。
　　　　　　- -p password：设置登录口令
　　　　　　- name：指定需要创建的用户名。  
　　2. usermod  
　　usermod修改一个用户的属性,usermod命令接的参数和useradd接的参数很多都是类似的。
　　　　- 使用语法`usermod [-option] name`
　　　　- 常用参数
　　　　　　- -a -G append group：添加用户所属分组,如果只有-G参数的话,该用户会从原来所属分组删除的。
　　3. userdel  
　　Userdel用来删除一个用户。  
　　　　- 使用语法 `userdel [-option] name`  
　　　　- 常用参数  
　　　　　　- -f force：强制删除
　　　　　　- -r remove：删除用户主目录下的文件以及主目录本身
　　4. passwd
　　修改用户的认证口令。
#### 添加、切换、删除用户组命令groupadd/newgrp/groupdel  
　　1. groupadd  
　　添加一个分组  
　　2. newgrp  
　　切换用户组,需要先用gpasswd为group添加密码。
　　3. groupdel  
　　删除一个用户组,若用户组下存在用户,需要先删除用户才能删除该组  
### 文件与目录的权限  
#### 查看文件与目录的权限ls -al  
　　例如:  

	[root@localhost c]#ls -al
	total 20
	drwxr-xr-x.  2 root root 4096 Mar 30 09:45 .
	drwxr-xr-x. 14 root root 4096 Apr  6 02:29 ..
	-rwxr-xr-x.  1 root root 6504 Mar 30 08:51 a.out
	-rw-r--r--.  1 root root  177 Mar 30 08:51 helloworld.c  
　　- 上面字段解释(图来自《鸟哥的linux私房菜》)
　　![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/authority.png)　　

　　- 权限字段解释  
　　![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/authority_properties.png)
#### 修改权限  
　　1. 改变所属群组, chgrp;分组必须存在。
　　`chgrp [-option] group filename/dirname`,  
　　2. 改变档案拥有者, chown;用户必须存在。  
　　`chown [-option] owner filename/dirname`  
　　3. 改变权限属性, chmod;  
　　　　- `chmod [-option] xyz filename/dirname`  

chmod | u<br>g<br>o<br>a | +(加入)<br>-(除去)<br>=(设定) | r<br>w<br>x | 档案或目录
--- | --- | --- | --- | ---
#### 目录权限属性  
　　1. r (read contents in directory)： 表示具有读取目录结构列表的权限，所以当您具有读取 (r) 一个目录的权限时， 您就可以利用 ls 这个指令将该目录的内容列表显示出来！  
　　2. w (modify contents of directory)： 他表示您将具有改动该目录结构列表的权限，也就是底下这些权限：  
　　　　- 建立新的档案与目录；  
　　　　- 删除已经存在的档案与目录(不论该档案是属于谁的！)  
　　　　- 将已存在的档案或目录进行更名；  
　　　　- 搬移该目录内的档案、目录位置。  
　　3. x (access directory)： 是否能将工作目录改为该目录。
### 参考资料  
　　《鸟哥的linux私房菜》
