title: php单例模式学习笔记
date: 2016-03-21 14:42:11
tags: [php,设计模式]
categories: 学习笔记

---
### 写在前面
　　关于单例模式和php单例模式的实现,网上已经有了很多的资料,这里有一些东西我就是从网上复制过来的,并注明了出处,在一些关键的地方,我也加上了自己的注解。  
### 单例模式  
#### 含义
　　在它的核心结构中只包含一个被称为单例的特殊类。通过单例模式可以保证系统中一个类只有一个实例而且该实例易于外界访问，从而方便对实例个数的控制并节约系统资源  
<!-- more -->
#### 使用单例模式的原因  
<blockquote>
PHP语言是一种解释型的脚本语言，这种运行机制使得每个PHP页面被解释执行后，所有的相关资源都会被回收。也就是说，PHP在语言级别上没有办法让某个对象常驻内存，这和asp.net、Java等编译型是不同的，比如在Java中单例会一直存在于整个应用程序的生命周期里，变量是跨页面级的，真正可以做到这个实例在应用程序生命周期中的唯一性。然而在PHP中，所有的变量无论是全局变量还是类的静态成员，都是页面级的，每次页面被执行时，都会重新建立新的对象，都会在页面执行完毕后被清空，这样似乎PHP单例模式就没有什么意义了，所以PHP单例模式我觉得只是针对单次页面级请求时出现多个应用场景并需要共享同一对象资源时是非常有意义的。
</blockquote>
#### 单例模式在PHP中的应用场合
　　1. 应用程序与数据库交互  
　　一个应用中会存在大量的数据库操作，比如过数据库句柄来连接数据库这一行为，使用单例模式可以避免大量的new操作，因为每一次new操作都会消耗内存资源和系统资源。  
　　2. 控制配置信息  
　　如果系统中需要有一个类来全局控制某些配置信息, 那么使用单例模式可以很方便的实现.
#### 三个要点+另外一个要点  
　　1. 需要一个保存类的唯一实例的静态成员变量:  

	private static $_instance;

　　2. 构造函数和克隆函数必须声明为私有的，防止外部程序new类从而失去单例模式的意义:  

	private function __construct()
	{
	
	}
	private function __clone()
	{
	
	}//覆盖__clone()方法，禁止克隆

　　3.  必须提供一个访问这个实例的公共的静态方法（通常为getInstance方法），从而返回唯一实例的一个引用:

	public static function getInstance()
	{
    	if(! (self::$_instance instanceof self) )
    	{
        	self::$_instance = new self();    
    	}  
    	return self::$_instance;    
	}   

　　4. 另外在鸟哥博客中有一篇文章讲解到了Serialize/Unserialize破坏单例的情况,这里是[链接](http://www.laruence.com/2011/03/18/1909.html),所以在构造单例的时候,还需要加上这两句,解决提到的问题:

	public  function __wakeup() 
	{
        self::$instance = $this;
    }
 
    /** 需要在单利切换的时候做清理工作 */
    public function __destruct() 
	{
        //只做清理工作.
		self::$instance = NULL;
    }

　　这里我写了一个单例的例子,MemcachedHandler::singleton()获取的是MemcachedHandler的实例,同时MemcachedHandler::singleton()->memcached可以获得Memcached的实例。
	class MemcachedHandler
	{
	    public $memcached;
	
	    private static $instance = null;
	
	    private function __construct()
	    {
	        $this->connect();
	    }
	
	    private function __clone()
	    {
	
	    }
	
	    public function __awake()
	    {
	        self::$instance = $this;
	    }
	
	    public function __destruct()
	    {
	        self::$instance = null;
	    }
	
	    public static function singleton()
	    {
	        if(!(self::$instance)){
	
	            self::$instance = new self();
	        }
	        return self::$instance;
	    }
	
	    private function connect()
	    {
	        try{
	            if(!class_exists('memcached'))
	            {
	                throw new Exception('memcached extension is needed!');
	            }
	            $this->memcached = new Memcached();
	
	            $config = include_once 'config.php';
	
	            if($this->memcached->addServer($config['memcached']['host'], $config['memcached']['port'], $config['redis']['weight']))
	            {
	                throw new Exception('memcached connection failed');
	            }
	
	        }
	        catch(Exception $e)
	        {
	            //log error
	            return false;
	        }
	    }
	}  
　　另外我又尝试了另外的一种写法,这里的单例主要是为了获取Memcached的实例,减少连接Memcached的次数,这种方式不像是一个真正的单例,因为继承了Memecached类,所以构造函数就不能声明为私有的了。  

	class MemcachedHandler extends Memcached
	{
		//保存memcahed实例
	    private static $instance;
		//获取实例
	    public static function singleton()
	    {
	        // TODO: Implement singleton() method.
	        try{
	            if(!class_exists('memcached'))
	            {
	                throw new Exception('memcached extension is needed!');
	            }
	            if(null === self::$instance){
	                self::$instance = new Memcached();
	                $config = include_once 'config.php';
	                if(self::$instance->addServer($config['memcached']['host'], $config['memcached']['port'], $config['redis']['weight']))
	                {
	                    throw new Exception('memcached connection failed');
	                }
	            }
	            return self::$instance;
	        }
	        catch(Exception $e)
	        {
	            //log error
	            return false;
	        }
	    }
	}
### 参考资料
　　[Serialize/Unserialize破坏单例](http://www.laruence.com/2011/03/18/1909.html)  
　　[可序列化单例模式的遗留问题答案](http://www.laruence.com/2011/03/18/1916.html)  
　　[PHP 单例模式解析和实战](http://blog.csdn.net/jungsagacity/article/details/7618587)