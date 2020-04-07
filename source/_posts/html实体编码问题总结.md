title: html实体编码问题总结
date: 2016-03-08 11:53:35
tags: [html,php,javascript]
categories: 学习笔记

---
### html字符实体  
　　在HTML中，有些些字符是预留的,有特殊意义的,所以这些字符是不允许在文本中使用的。比如小于号（<）和大于号（>）。如果希望正确地显示预留字符，我们必须HTML 源代码中使用字符实体（character entities）。  
　　要转换html实体的原因主要是在html页面中有一些字符实体如果不转换的话会对本身的html页面产生影响，这里面主要就有`<`,`>`,`"`,`'`,空格等。
　　下面是一些常见的HTML字符实体  

显示结果 | 描述 | 实体名称 | 实体编号
--- | --- | --- | ---
 | 空格 | `&nbsp;` | `&#160;`
< | 小于号 | `&lt;` | `&#60;`
> | 大于号 | `&gt;` | `&#62;`
& | 和号 | `&amp;` | `&#38;`
" | 双引号 | `&quot;` | `&#34;`
' | 单引号 | `&apos;` | `&#39;`
′ | 重音符 | `&acute;` | `&#96;`
© | 版权 | `&copy;` | `&#169;`
® | 注册商标 | `&reg;` | `&#174;`
™ | 商标 | `&trade;` | `&#8482;`  
　　在实际的工作中经常会遇到要处理HTML字符实体的情况，这里我总结了一些处理方法。  
<!-- more -->
### php  
　　1. htmlspecialchars()  
　　htmlspecialchars() 函数把预定义的字符转换为 HTML 实体。  
　　预定义的字符包括：`&`,`"`,`'`,`<`,`>`  


参数 | 必需性 | 描述
--- | --- | ---
string | 必需 | 规定要转换的字符串
quotestyle | 可选 | 规定如何编码单引号和双引号<br>ENT_COMPAT - 默认。仅编码双引号。<br>ENT_QUOTES - 编码双引号和单引号。<br>ENT_NOQUOTES - 不编码任何引号。
character-set | 可选 | 字符串值，规定要使用的字符集。
double_encode | 可选 | 布尔值。规定了是否编码已存在的 HTML 实体。

提示和注释
提示：在 PHP5.4 之前的版本，无法被识别的字符集将被忽略并由 ISO-8859-1 替代。自 PHP5.4 起，无法被识别的字符集将被忽略并由 UTF-8 替代。 character-set 自5.6版本开始默认使用的字符集可以在设置项中改变。对应的解码函数htmlspecialchars_decode()。  
　　2. htmlentities()  
　　这个函数的参数是和htmlspecialchars()完全相同的,它们的区别就在于htmlspecialchars()只会转换特定的字符,就是上面所说的预定义的字符;而htmlentities()会转换所有的html字符实体;对应的解码函数html_entity_decode()。
### javascript  
　　1. 利用正则匹配转换  
<code>
	function html_encode(str)  
	{  
	　　var  s    =    `""`;  
	　　if    (str.length == 0)  return `""`;  
	　　s    =    str.replace(/`&`/g,`"&gt;"`);  
	　　s    =    s.replace(/`<`/g,`"&lt;"`);  
	　　s    =    s.replace(/`>`/g,`"&gt;"`);  
	　　s    =    s.replace(/&nbsp;/g,`"&nbsp;"`);  
	　　s    =    s.replace(/`\'`/g,`"&apos;"`);  
	　　s    =    s.replace(/`\"`/g,`"&quot;"`);  
	　　s    =    s.replace(/`\n`/g,`"<br>"`);  
	return    s;  
	}  
	function html_decode(str)  
	{  
	　　var    s    =    `""`;  
	　　if    (str.length==0)    return    `""`;  
	　　s    =    str.replace(/`&gt;`/g,`"&"`);  
	　　s    =    s.replace(/`&lt;`/g,`"<"`);  
	　　s    =    s.replace(/`&gt;`/g,`">"`);  
	　　s    =    s.replace(/`&nbsp;`/g,`"`&nbsp;`"`);  
	　　s    =    s.replace(/`&apos;`/g,`"\'"`);  
	　　s    =    s.replace(/`&quot;`/g,`"\""`);  
	　　s    =    s.replace(/`<br>`/g,`"\n"`);  
	　　return    s;  
	}  
</code>
　　2. 用的浏览器内部转换器实现转换，方法是动态创建一个容器标签元素，如DIV，将要转换的字符串设置为这个元素的innerText，然后返回这个元素的innerHTML，即得到经过HTML编码转换的字符串。  
<code>
	function html_encode(html)  
	{  
	　　return document.createElement(`'div'`)  
	　　.appendChild(document.createTextNode(html))  
	　　.parentNode.innerHTML;  
	}  
	function html_decode(html)  
	{  
	　　var a = document.createElement(`'div'`);  
	　　a.innerHTML = html;  
	　　return a.textContent;  
	}  
</code>
　　这种方法有一个问题就是对于单引号`'`和双引号`"`不会进行转义处理,对于这种方法使用的时候就要非常注意了。  
　　比如转义<a href="http://blog.owlslink.xyz"></a>,用第一种方法:  
　　`&lt;a href=&quot;http://blog.owlslink.xyz&quot;&gt;&lt;/a&gt;`  
　　第二种方法:  
　　`&lt;a href="http://blog.owlslink.xyz"&gt;&lt;/a&gt;`  
　　虽然第二种方法看起来简洁,但是在一些特殊情况下是会出现问题的,总之我还是比较倾向于使用第一种用正则匹配的方法,根据实际情况可以自己控制需要转换的实体。