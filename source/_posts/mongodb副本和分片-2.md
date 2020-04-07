title: mongodb副本和分片(2)
date: 2017-03-09 11:07:51
tags: [mongo]
categories : 学习笔记

---
### 安装
#### 安装版本
　　mongodb3.4.2
#### systemctl脚本

		[Unit]
		Description=mongod service
		
		[Service]
		User=root
		ExecStart=/usr/local/mongodb3.4.2/bin/mongod --config /data/db/mongo.conf
		ExecStop=/usr/local/mongodb3.4.2/bin/mongod --shutdown
		Type=notify
		Restart=always
		KillSignal=SIGQUIT
		StandardError=syslog
		NotifyAccess=all
		
		[Install]
		WantedBy=multi-user.target

### 配置
#### 常规配置
		systemLog:
		    destination: file
		    path: /var/log/mongo/mongo.log
		    traceAllExceptions: true
		
		processManagement: 
		    fork: true
		    pidFilePath: /data/db/mongo.pid
		
		net:
		    port: 27017
		    maxIncomingConnections: 2048
		
		storage:
		    dbPath: /data/db

#### 分片
#### 副本集
### 参考资料
　　[Mongo Configuration File Options](https://docs.mongodb.com/manual/reference/configuration-options/)  
　　[Mongo Sharding](https://docs.mongodb.com/manual/sharding/)  
　　[Mongo Replication](https://docs.mongodb.com/manual/replication/)  
　　[install-mongodb-on-ubuntu](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)

