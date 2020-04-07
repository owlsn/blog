title: ES集群
date: 2017-03-09 15:55:15
tags: [elasticsearch]
categories: 学习笔记

---
### 概念

### 安装配置
	# 安装java环境
	sudo apt-get update
	sudo apt-get install openjdk-8-jdk
	# 配置环境变量
	sudo bash -c 'echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> /etc/profile'
	sudo bash -c "source /etc/profile"
	# 下载elasticsearch
	sudo wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.1.zip
	sudo unzip elasticsearch-5.4.1.zip
	# 修改配置
	# 2g默认jvm运行内存,按实际情况修改
	sudo sed -i 's/Xms2g/Xms1g/g' elasticsearch-5.4.1/config/jvm.options
	sudo sed -i 's/Xmx2g/Xmx1g/g' elasticsearch-5.4.1/config/jvm.options
	sudo sed -i 's/^\#network.host.*/network.host:\ 0.0.0.0/g'  elasticsearch-5.4.1/config/elasticsearch.yml
	sudo bash -c "echo 'vm.max_map_count=262144' >> /etc/sysctl.conf"
	sudo sysctl -p
	# 开放端口
	sudo ufw allow 9200
	# 添加用户和用户组
	sudo groupadd elsearch
	sudo useradd -g elsearch elsearch
	# 改变目录权限
	sudo chown -R elsearch:elsearch elasticsearch-5.4.1
	# 以elsearch用户启动
	sudo su - elsearch -c elasticsearch-5.4.1/bin/elasticsearch

　　

	# 集群配置
	
	/usr/local/elasticsearch-5.4.1/bin/elasticsearch -Epath.conf=/usr/local/elasticsearch-5.4.1/config/config-master-02 -d
　　

	# _cat API 查看集群状态
	curl -v -XGET http://localhost:9200/_cat/
	# 创建索引
	curl -v -XPUT http://localhost:9200/customer
	# 查看索引
	curl -v XGET　http://localhost:9200/_cat/indices?v
	# 指定id写入数据
	curl -v -XPUT http://localhost:9200/customer/external/1 -d '{"name" : "owlsn"}'
	# 查询数据
	curl -v -XGET http://localhost:9200/customer/external/1?pretty
	# 不指定id写入
	curl -v -XPOST http://localhost:9200/customer/external -d '{"name" : "owlsn"}'
	# 更新数据
	curl -v -XPOST http://localhost:9200/customer/external/1/_update -d '{"doc":{"name" : "owlsn"}}'
	# 删除索引
	curl -v -XDELETE http://localhost:9200/customer
	# restful API 格式
	<REST Verb> /<Index>/<Type>/<ID>
	# bulk API 批量操作
	curl -v -XPOST http://localhost:9200/customer/external/_bulk -d '
	{"update":{"_id":1}}
	{"doc":{"name":"test"}}
	{"delete":{"_id":1}}'

　　

	#  

### 参考资料
　　[Elasticsearch: 权威指南](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/index.html)
