环境：我们使用之前部署的zookeeper集群，因为Kafka服务依赖于zookeeper服务。
三个节点： IP地址		主机名			节点
	192.168.200.169		zookeeper1		集群节点
	192.168.200.170		zookeeper2		集群节点
	192.168.200.171		zookeeper3		集群节点

所需软件包：kafka_2.11-1.1.1.tgz				//这里提供下载地址：http://kafka.apache.org/downloads.html

1：解压缩 Kafka 软件包（3个节点操作）
[root@zookeeper1 ~]# tar -zxvf kafka_2.11-1.1.1.gz	//解压到 /root下

2：修改3个基点配置文件
[root@zookeeper1 ~]# vi kafka_2.11-1.1.1/config/server.properties	//进入配置文件
#找到下列两行，并注释掉
broker.id=0
zookeeper.connect=localhost:2181
#然后