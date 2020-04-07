title: php工厂模式学习笔记
date: 2016-03-20 17:37:50
tags: [php,设计模式]
categories: 学习笔记

---
### 写在前面  
　　主要内容:工厂模式要点,用工厂模式实现sessionHandler  
### 工厂模式  
　　工厂类就是一个专门用来创建其它对象的类，工厂类在多态性编程实践中是非常重要的。它允许动态替换类，修改配置，会使应用程序更加灵活。工厂模式的最主要作用就是对象创建的封装、简化创建对象操作。 简单的说，就是调用工厂类的一个方法（传入参数）来得到需要的类
　　工厂模式我能想到的典型的应用就是配置数据库连接或者缓存连接。
### 示例  
　　这里我用redis和memecached作session存储的例子。这里同时还把单例模式结合进去了。
　　1. 简单工厂模式  

	//通过静态方法创建对象
	//抽象接口
	interface HandlerInterface
	{
    	public function singleton();	
	}
<br>

	//继承接口的memcachedHander类
	class MemcachedHandler extends Memcached implements HandlerInterface
	{
		//保存memcahed实例
	    private static $instance;
		//获取实例
	    public function singleton()
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
	//和上面类似的,继承接口的redisHandler类
	class RedisHandler extends Redis implements HandlerInterface
	{
		//保存实例
	    private static $instance;
		//获取实例
	    public function singleton()
	    {
	        // TODO: Implement singleton() method.
	        try{
	            if(!class_exists('redis'))
	            {
	                throw new Exception('redis extension is needed!');
	            }
	            if(null === self::$instance){
	                self::$instance = new Redis();
	                $config = include_once 'config.php';
	                if(self::$instance->connect($config['redis']['host'], $config['redis']['port'], $config['redis']['lifetime']))
	                {
	                    throw new Exception('redis connection failed');
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
<br>

	//工厂类,
	class HandlerFactory
	{
	    public static function getHander($type)
	    {
	        switch($type)
	        {
	            case 'redis' :
	                return new RedisHandler();
	            case 'memcached' :
	                return new MemcachedHandler();
	        }
	    }
	}
<br>

	//最后工厂产生redis单例
	$sessionHandler = HandlerFactory::getHander('redis')->singleton();
　　