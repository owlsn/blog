title: php篇-include_path
date: 2016-04-4 21:17:28
tags: [php]
categories: 学习笔记

---
### include_path  
　　[include_path](http://php.net/manual/zh/ini.core.php#ini.include-path),include_path参数指定的目录,fopen(), file(), readfile(), file_get_contents()这些函数在查找文件时会参照这些目录。参数目录当然也要按照一定的格式要求，在php.ini文件中有例子说明:  
<blockquote>
; UNIX: "/path1:/path2"  
;include_path = ".:/php/includes"
;
; Windows: "\path1;\path2"
;include_path = ".;c:\php\includes"
</blockquote>
<!-- more -->
　　并且include_path参数可以设置不止一个目录，如果设置了多个目录的话，在查找文件时候会在这些设置的目录中逐个寻找。  
　　另外,在程序运行中也可以通过set_include_path()函数改变include_path()的参数。  
### set_include_path()和ini_set('include_path', '')  
　　set_include_path()函数能在程序运行中改变include_path()的值,这个函数对于动态加载一些文件是非常有用处的。ini_set()函数也能改变include_path的值,相比较而言,ini_set()更通用一些。设置的时候,unix中分隔符是`:`,而windows中是`;`,在php中用常量PATH_SEPARATOR统一代替。  
　　一般情况下在程序中动态改变include_path只是在php.ini设置的基础上增加路径,即先用`ini_get('include_path')`获取原本的参数,然后将增加的参数添加进去。  
<code>  
ini_set(  
　`'include_path'`,  
　　__SYSDIR__ . `'include' `. PATH_SEPARATOR  
　　. ini_get(`'include_path'`)  
);  
</code>  
　　上文中的__SYSDIR__是根据实际情况相对于根目录设置的,这样设置的情况下就可以将一些配置文件都放在相应的目录中,当整个项目移动时也可以不用修改少量的路径配置了。  
　　另外路径分隔符`/`在php注意用`DIRECTORY_SEPARATOR`代替
### include和require  
　　include用法参见[php手册-include](http://php.net/manual/zh/function.include.php)  
　　这里我自己找了几个需要注意的地方:  
　　1. require/include顺序问题,这个问题在鸟哥的blog中有非常详细的解答,鸟哥的blog[深入理解PHP之require/include顺序](http://www.laruence.com/2010/05/04/1450.html),影响顺序的因素主要是参数的格式(是路径还是文件名,是绝对路径还是相对路径)。
　　2. 如果没有找到目标文件,include发出警告,后面的程序会继续执行,而require则是导致一个致命错误,使得程序终止运行。
　　3. 变量作用域问题,在被包含文件中定义的变量继承include所在行的变量范围,而定义的函数和类不同,都具有全局作用域。  
　　4. require_once,include_once会检测文件是否被包含过,如果是则不会再次包含  
　　5. require是无条件加载,在解析程序时就会加载文件,而include是条件加载，当程序执行到了include语句处才会根据条件选择加载。
### 参考资料  
　　鸟哥的blog[深入理解PHP之require/include顺序](http://www.laruence.com/2010/05/04/1450.html)
