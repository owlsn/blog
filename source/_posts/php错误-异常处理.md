title: php错误/异常处理
date: 2016-03-25 15:17:15
tags: [php]
categories: 学习笔记

---
### 写在前面  
　　主要内容:错误类型,调试方法,自定义错误处理函数set_error_handler(),错误日志处理,异常处理。
### 错误类型  
　　运行PHP脚本时，PHP解析器会尽其所能地报告它遇到的问题。在PHP中错误报告的处理行为，都是通过PHP的配置文件php.ini中有关的配置指令确定的。另外PHP的错误报告有很多种级别，可以根据不同的错误报告级别提供对应的调试方法。下表列出了php中大多数的错误报告级别：这里我只列举了一部分(具体所有常量参见php手册[错误处理预定义常量](http://php.net/manual/zh/errorfunc.constants.php))

值 | 常量 | 说明 | 备注
-- | -- | -- | --
1 | E_ERROR (integer) | 致命的运行时错误。这类错误一般是不可恢复的情况，例如内存分配导致的问题。后果是导致脚本终止不再继续运行。
2 | E_WARNING (integer) | 运行时警告 (非致命错误)。仅给出提示信息，但是脚本不会终止运行。
4 | E_PARSE (integer) | 编译时语法解析错误。解析错误仅仅由分析器产生。
8 | E_NOTICE (integer) | 运行时通知。表示脚本遇到可能会表现为错误的情况，但是在可以正常运行的脚本里面也可能会有类似的通知。
16 | E_CORE_ERROR (integer) | 在PHP初始化启动过程中发生的致命错误。该错误类似 E_ERROR，但是是由PHP引擎核心产生的。 | since PHP 4
32 | E_CORE_WARNING (integer) | PHP初始化启动过程中发生的警告 (非致命错误) 。类似 E_WARNING，但是是由PHP引擎核心产生的。 | since PHP 4
64 | E_COMPILE_ERROR (integer) | 致命编译时错误。类似E_ERROR, 但是是由Zend脚本引擎产生的。 | since PHP 4
128 | E_COMPILE_WARNING (integer) | 编译时警告 (非致命错误)。类似 E_WARNING，但是是由Zend脚本引擎产生的。 | since PHP 4
30719 | E_ALL (integer) | E_STRICT出外的所有错误和警告信息。 | 30719 in PHP 5.3.x, 6143 in PHP 5.2.x, 2047 previously  
　　设置出错时候是否输出错误消息:  
　　1. php.ini文件中的display_errors参数(on-开启，off-关闭)
　　2. 运行中动态设置,ini_set('display_errors', true)  
　　在程序开发阶段打开错误消息输出,方便调试；线上环境关闭,防止泄露服务器相关的信息,使服务器变得不安全。  
　　设置出错时错误报告的级别:
　　1. php.ini文件中的error_reporting参数,error_reporting=E_ALL & ~E_NOTICE
　　2. 运行中动态设置,error_reporting(E_ALL & ~E_NOTICE),
### php错误日志  
　　产品投入运营之后,线上环境就会关闭错误消息输出,以免因为这些错误所透露的路径、数据库连接、数据表等信息而遭到黑客攻击。那么就需要错误日志来记录程序运行中产生的错误信息。
　　1. 开启php错误日志记录,在php.ini中设置log_errors参数on,那么在程序中产生的php错误就会记录到服务器的错误日志之中。  
　　2. 设置了error_log参数,就可将错误日志记录到指定路径的文件之中
　　3. php中的error_log()函数,可以送出用户自定义的消息到日志文件,邮件,发送到其他服务器或者到操作系统的日志。
### [set_error_handler()](http://php.net/manual/zh/function.set-error-handler.php)  
　　set_error_handler(PHP 4 >= 4.0.1, PHP 5, PHP 7) — 设置一个用户定义的错误处理函数  
　　函数原型:  
　　`mixed set_error_handler ( callable $error_handler [, int $error_types = E_ALL | E_STRICT ] )`  
　　1. 设置用户自定义函数error_handler处理脚本中的错误,用户自定义函数原型：  
　　　　`handler ( int $errno , string $errstr [, string $errfile [, int $errline [, array $errcontext ]]] )`
　　　　- 错误码errno和描述错误的errstr是必选参数,另外有可能提供三个可选参数：发生错误的文件名errfile、发生错误的行号errline 以及发生错误的上下文errcontext(一个指向错误发生时活动符号表的 array)。   
　　　　- 如果函数返回 FALSE，标准错误处理处理程序将会继续调用。
　　　　- 如果错误发生在脚本执行之前，将不会调用自定义的错误处理程序因为它尚未在那时注册。
　　　　- 自定义函数处理完成之后程序会继续执行。
　　例子：  

	//先定义一个函数，也可以定义在其他的文件中，再用require()调用  
	function myErrorHandler($errno, $errstr, $errfile, $errline)  
	{  
		//为了安全起见，不暴露出真实物理路径，下面两行过滤实际路径  
    	$errfile=str_replace(getcwd(),"",$errfile);  
    	$errstr=str_replace(getcwd(),"",$errstr);  
  
    	switch ($errno) {  
    	case E_USER_ERROR:   
     		echo "<b>My ERROR</b> [$errno] $errstr<br />\n";  
        	echo "  Fatal error on line $errline in file $errfile";  
        	echo ", PHP " . PHP_VERSION . " (" . PHP_OS . ")<br />\n";  
        	echo "Aborting...<br />\n";  
        	exit(1);  
        	break;  
    	case E_USER_WARNING:  
        	echo "<b>My WARNING</b> [$errno] $errstr<br />\n";  
        	break;    
    	case E_USER_NOTICE:  
        	echo "<b>My NOTICE</b> [$errno] $errstr<br />\n";  
        	break;  
    	default:  
        	echo "Unknown error type: [$errno] $errstr<br />\n";  
        	break;  
    	}  
    /* Don't execute PHP internal error handler */  
    return true;  
	}  
　　2. error_types  此参数能够用于屏蔽 error_handler 的触发,即此参数的设置是产生错误时出发error_handler函数的错误级别。 如果没有该掩码(默认值是E_ALL | E_STRICT)，也就是说无论 error_reporting 是如何设置的， error_handler 都会在每个错误发生时被调用。
　　3. 如果之前有定义过错误处理程序，则返回该程序名称的 string；如果是内置的错误处理程序，则返回 NULL。 如果你指定了一个无效的回调函数，同样会返回 NULL。 如果之前的错误处理程序是一个类的方法，此函数会返回一个带类和方法名的索引数组(indexed array)。
　　三种用法:  

	class CallbackClass {
	   function CallbackFunction() {
    	   // refers to $this
	   	}
	   function StaticFunction() {
    	   // doesn't refer to $this
		}
	}

	function NonClassFunction($errno, $errstr, $errfile, $errline) {
	}

	1: set_error_handler('NonClassFunction');  // 直接转到一个普通的函数 NonClassFunction

	2: set_error_handler(array('CallbackClass', 'StaticFunction')); // 转到 CallbackClass 类下的静方法 StaticFunction

	3: $o =& new CallbackClass();
    set_error_handler(array($o, 'CallbackFunction'));  // 转到类的构造函数，其实本质上跟下面的第四条一样。

	4: $o = new CallbackClass();
	// The following may also prove useful:
	class CallbackClass {
		function CallbackClass() {
    	   set_error_handler(array(&$this, 'CallbackFunction')); // the & is important
	   }
	   function CallbackFunction() {
	       // refers to $this
	   }
	}  
### try catch  
　　异常（Exception）处理用于在指定的错误发生时改变脚本的正常流程，是PHP 5中的一个新的重要特性。异常处理是一种可扩展、易维护的错误处理统一机制，并提供了一种新的面向对象的错误处理方式。  
　　实例：  

	try {
		$error = 'Always throw this error';
        throw new Exception($error);      //创建一个异常对象，通过throw语句抛出
        echo 'Never executed';            //从这里开始，try代码块内的代码将不会再被执行
    } catch (Exception $e) {
        echo 'Caught exception: ',  $e->getMessage(), "\n";  //输出捕获的异常消息
    }
    echo 'Hello World';                    //程序没有崩溃继续向下执行
　　1. 在某些编程语言中，例如Java，在出现异常时将自动抛出异常。而在PHP中，异常必须手动抛出
　　2. 扩展内置Exception类  

	class Exception {
		protected $message = 'Unknown exception';              //异常信息
        protected $code = 0;                              //用户自定义异常代码
        protected $file;                                   //发生异常的文件名
        protected $line;                                  //发生异常的代码行号
        public function __construct($message = "", $code = 0, Exception $previous = null) {}    //构造方法
		/*
		*其中还有一些其他的方法,根据版本不同
		*/
        final function getMessage(){}                        //返回异常信息
        final function getCode(){}                           //返回异常代码
        final function getFile(){}                            //返回发生异常的文件名
        final function getLine(){}                            //返回发生异常的代码行号
        final function getTrace(){}                           //backtrace() 数组
        final function getTraceAsString(){}                    //已格成化成字符串的 getTrace() 信息
        /* 可重载的方法 */
        function __toString(){}                             //可输出的字符串
        }  
　　扩展的时候注意构造方法和__toString(){}方法,另外还可以加入自己的自定义方法。
### 参考资料  
　　http://jingyan.baidu.com/article/046a7b3ed5e233f9c37fa969.html  
　　http://www.cnblogs.com/laojie4321/p/4187620.html  
　　http://justcoding.iteye.com/blog/848711