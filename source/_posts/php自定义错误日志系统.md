title: php自定义错误日志系统
date: 2016-03-26 17:00:14
tags: [php]
categories: 学习笔记

---
### 写在前面  
　　主要内容:monolog,mongodb,set_error_handler   
　　monolog是一个功能非常强大的日志类库,而且扩展性非常好,这篇文章主要就是记录了我自己配置monolog使用mongodb存储日志的一些步骤,另外还写了我自己看monolog源码学到的一些东西。  
### 使用monolog
#### 引入monolog  
　　- 使用composer加载monolog  

	"require": {
        "monolog/monolog": "1.0.*"
    }  
　　- 加载autoload  

	require __SYSDIR__.'/vendor/autoload.php';  
　　- 
#### 核心概念  
　　1. Logger实例可以定义自己的频道名和handler堆栈,handler可以在多个Logger中共享。频道名会反映在日志里，方便我们查看和过滤日志记录。
　　2. channel频道,使用频道名可以对日志进行分类，这在大型的应用上是很有用的。通过频道名，可以很容易的对日志记录进行刷选。  
　　例如我们想在同一个日志文件里记录不同模块的日志，我们可以把相同的handler绑定到不同的Logger实例上，这些实例使用不同的频道名。
　　3. handler是真正进行日志操作的实体,monolog有一些内置的handler：
　　　- StreamHandler：把记录写进PHP流，主要用于日志文件。
　　　- SyslogHandler：把记录写进syslog。
　　　- ErrorLogHandler：把记录写进PHP错误日志。
　　　- RedisHandler：把记录写进Redis。
　　　- MongoDBHandler：把记录写进Mongo。
　　　- BufferHandler：允许我们把日志记录缓存起来一次性进行处理。
　　另外还有一些内置的handler可以在[github/monolog](https://github.com/Seldaek/monolog/blob/master/doc/02-handlers-formatters-processors.md)上找到更详细的描述。  
　　如果monolog内置的handler不能满足要求,我们还可以自己自定义handler去处理日志信息,一般需要继承Monolog\Handler\HandlerInterface接口,另外,monolog里面有一个继承了Monolog\Handler\HandlerInterface接口的抽象类AbstractProcessingHandler,我们也可以自定义handler继承这个抽象类,可以不用一个个去实现HandlerInterface里的所有方法。
　　4. Formatters表示日志写入文件,邮件,数据库,socket或者其他handler的格式。
　　5. processor可以为日志记录添加额外的信息
　　所有关于monolog的参考资料都可以在github上找到,这里是[链接](https://github.com/Seldaek/monolog/)
#### 封装日志类  
　　1. 要使用mongodb存储日志,首先封装一个mongodb单例  

	class MongoLog
	{
	    private static $instance;
	    private function __construct()
	    {
	        $this->connect();
	    }
	    private function __clone()
	    {
	
	    }
	    public function __destruct()
	    {
	        self::$instance = null;
	    }
	    public function __wakeup()
	    {
	        self::$instance = $this;
	    }
	    public static function getInstance()
	    {
	        if(null === self::$instance)
	        {
	            self::$instance = new self();
	        }
	        return self::$instance;
	    }
	
	    public function connect()
	    {
	        
	    }
	}  
　　2. 定义错误等级常量  

	const EMERG = 600;  // Emergency: system is unusable
    const ALERT = 550;  // Alert: action must be taken immediately
    const CRIT = 500;  // Critical: critical conditions
    const ERR = 400;  // Error: error conditions
    const WARN = 300;  // Warning: warning conditions
    const NOTICE = 250;  // Notice: normal but significant condition
    const INFO = 200;  // Informational: informational messages
    const DEBUG = 100;  // Debug: debug messages
　　3. 在connect方法中封装连接mongo的方法,以及初始化一些mongo的配置。首先定义一些私有属性存储这些配置信息。  

	private $mongo;      //mongo实例
    private $db;         //mongo数据库
    private $tag;        //日志标志,channel
    private $type;       //日志类型
    private $collection;     //日志集合,mongo collection  

	public function connect()
    {
        $configs = C('db_config', 'mongodb');
        if(!$configs)
        {
            throw new UnexpectedValueException('mongo config is empty');
        }

        $this->mongo = new MongoClient($configs['server']);

        $this->db = $configs['db'] ?$configs['db']: 'mongo_logs';

        $this->tag = $configs['tag'] ?$configs['tag']: 'error_log';

        $this->collection = $configs['collection'] ?$configs['collection']: 'log';

        $this->type = $configs['type'] ?$configs['type']: self::INFO;
    }
　　4. 定义一些设置方法,供自定义这些mongo配置属性,并提供链式调用。  

	public function setDb($db = 'mongo_logs')
    {
        $this->db = $db;
        return $this;
    }
    public function setTag($tag = 'error_log')
    {
        $this->tag = $tag;
        return $this;
    }
    public function setCollection($collection = 'log')
    {
        $this->db = $collection;
        return $this;
    }
    public function setType($type = self::INFO)
    {
        $this->db = $type;
        return $this;
    }
　　5. 定义log方法,实例化monolog mongoDBHandler,并加入一些其他的日志信息,用私有属性$logger保存logger实例,并定义获取logger实例的方法。  

	private $logger;

	public function getLogger()
    {
        return $this->logger;
    }

    public function log($message, array $context = array())
    {
        $log = new Logger($this->tag);
        $this->logger = $log;
        $log->pushProcessor(
            function ($record)
            {
                $env = array(
                    'url'         => sprintf('http://%s%s', $_SERVER['HTTP_HOST'], $_SERVER['REQUEST_URI']),
                    'ip'          => get_client_ip(),
                    'http_method' => $_SERVER['REQUEST_METHOD'],
                    'referrer'    => $_SERVER['HTTP_REFERER'],
                    'file'        => $_SERVER['SCRIPT_NAME'],
                );
                $record['extra'] = array_merge($record['extra'], $env);
                return $record;
            }
        );

        $log->pushHandler(new MongoDBHandler($this->mongo, $this->db, $this->collection));
        switch($this->type)
        {
            case self::DEBUG :
                return $log->addDebug($message, $context);
            case self::ERR :
                return $log->addError($message, $context);
            case self::WARN :
                return $log->addWarning($message, $context);
            default:
                return $log->addInfo($message, $context);

        }
    }  
　　这样一个基本的mongo日志处理类就封装完成了,当然其中可能有一些疏漏,但是大致的思路差不多就是这样的了。  
　　6. 最后调用的方式就是  

	MongoLog::getInstance()
            ->setTag($channel)
            ->setType($type)
            ->log($msg, $context);
　　这里我们还可以封装成一个方法  

	function writeLog($msg, $type, $channel = 'error_log', array $context = array())
	{
	    try
	    {
	        $res = MongoLog::getInstance()
            ->setTag($channel)
            ->setType($type)
            ->log($msg, $context);
	        if(!$res){
	            throw new UnexpectedValueException('UnexpectedValueException:log error');
	        }
			return true;
	    }
	    catch(Exception $e)
	    {
	        //write file log
	    }
	}  
　　当发生错误的时候就在catch中写入文件日志。  
　　7. 另外monolog的\Monolog\ErrorHandler::register()方法可以通过传入logger实例注册set_error_handler(),set_exception_handler(),register_shutdown_function()方法来接管php自身的错误信息处理,所以我们在上面的$logger属性中保存了logger实例,所以我们可以将之传入\Monolog\ErrorHandler::register()方法中,使用mongo记录php的错误日志。
### 参考资料  
　　[Monolog - Logging for PHP 5.3+](https://segmentfault.com/a/1190000002775923?utm_source=tuicool&utm_medium=referral)  
　　[monolog](https://github.com/Seldaek/monolog/blob/master/doc/01-usage.md)