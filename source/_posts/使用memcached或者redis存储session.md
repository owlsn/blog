title: 使用memcached或者redis存储session
date: 2016-03-15 16:58:34
tags: [php,session,memcached,redis]
categories: 学习笔记

---
### 写在前面
　　主要内容:memcached,redis的php操作函数,session_set_save_handler,SessionHandlerInterface
### session多服务器共享  
　　这里我找到了一篇讲解这方面非常详细的文章-[session多服务器共享的方案梳理](http://www.cnblogs.com/wangtao_20/p/3395518.html),里面说到的几个方案我这里简单列举一下:  
　　1. 把原来存储在服务器磁盘上的session数据存储到客户端的cookie中去。
　　　　- 缺点:cookie大小有限,传送cookie占用了更大的带宽
　　2. sticky模式(粘性会话模式)，同一个用户的访问请求都被派送到同一个服务器上
　　　　- 缺点:和服务器负载均衡冲突
　　3. session存储层,即将所有的session统一管理
　　　　- 实现方式1磁盘共享
　　　　- 实现方式2数据库共享
　　　　- 实现方式3memcached或者redis缓存
　　4. session复制
　　　　- 缺点:速度慢,延迟高,带宽消耗过多
### redis或者memcached
　　使用redis或者memcached存储session有两种方法:  
　　1. 使用session_set_save_handler自定义处理函数
　　2. 使用php扩展提供的自带的sessionhandler(只针对某些扩展,比如memcached或者redis)  
　　针对1方法,php5.4之前是  
　　`bool session_set_save_handler ( callable $open , callable $close , callable $read , callable $write , callable $destroy , callable $gc [, callable $create_sid ] )`  
　　php5.4及之后可以直接实现 SessionHandlerInterface 接口  
　　`bool session_set_save_handler ( SessionHandlerInterface $sessionhandler [, bool $register_shutdown = true ] )`
　　对于SessionHandlerInterface,  

	SessionHandlerInterface {
	/* 方法 */
		abstract public bool close ( void )  //close 回调函数类似于类的析构函数。 在 write 回调函数调用之后调用。 当调用 session_write_close() 函数之后，也会调用 close 回调函数
		abstract public bool destroy ( string $session_id )  //当调用 session_destroy() 函数， 或者调用 session_regenerate_id() 函数并且设置 destroy 参数为 TRUE 时， 会调用此回调函数
		abstract public bool gc ( int $maxlifetime )  //为了清理会话中的旧数据，PHP 会不时的调用垃圾收集回调函数
		abstract public bool open ( string $save_path , string $name ) //open 回调函数类似于类的构造函数， 在会话打开的时候会被调用。 这是自动开始会话或者通过调用 session_start() 手动开始会话 之后第一个被调用的回调函数
		abstract public string read ( string $session_id )  //如果会话中有数据，read 回调函数必须返回将会话数据编码（序列化）后的字符串。 如果会话中没有数据，read 回调函数返回空字符串。
		abstract public bool write ( string $session_id , string $session_data )  //在会话保存数据时会调用 write 回调函数。 此回调函数接收当前会话 ID 以及 $_SESSION 中数据序列化之后的字符串作为参数。 序列化会话数据的过程由 PHP 根据 session.serialize_handler 设定值来完成。序列化后的数据将和会话 ID 关联在一起进行保存。 当调用 read 回调函数获取数据时，所返回的数据必须要和 传入 write 回调函数的数据完全保持一致。
	}  
　　这里实现使用缓存自定义存储session操作的方法我以laravel框架中的CaseBaseSessionHandler举例  

	class CacheBasedSessionHandler implements SessionHandlerInterface {
		protected $cache;　　//The cache repository instance.
		protected $minutes;		//The number of minutes to store the data in the cache.

		/**
		 * Create a new cache driven handler instance.
		 */
		public function __construct(CacheContract $cache, $minutes)
		{
			$this->cache = $cache;
			$this->minutes = $minutes;
		}
	
		public function open($savePath, $sessionName)
		{
			return true;
		}
	
		public function close()
		{
			return true;
		}
	
		public function read($sessionId)
		{
			return $this->cache->get($sessionId, '');
		}
	
		public function write($sessionId, $data)
		{
			return $this->cache->put($sessionId, $data, $this->minutes);
		}
	
		public function destroy($sessionId)
		{
			return $this->cache->forget($sessionId);
		}
	
		public function gc($lifetime)
		{
			return true;
		}
	
		public function getCache()
		{
			return $this->cache;  //Get the underlying cache repository.
		}

	}
　　在构造函数中的CacheContract可以先简单的理解为一个接口,我们可以另外根据使用的是redis还是memcached来实现这个接口,这个接口里面包含了一些对于缓存的操作。  
　　针对方法2,可以在php.ini中配置session.save_handler参数,或者在程序中  
　　`ini_set("session.save_handler", memcached);`  
　　同时,还需要设置session.save_path参数为缓存的地址和端口,可以在php.ini中设置或者在程序中  
　　`ini_set(session.save_path", "tcp://localhost:6379");`  
### 参考资料
　　[session多服务器共享的方案梳理](http://www.cnblogs.com/wangtao_20/p/3395518.html)  
　　[session放入缓存（redis）、DB](http://www.tuicool.com/articles/yeeyume)  
　　[laravel-session](https://laravel.com/docs/5.2/session#adding-custom-session-drivers)