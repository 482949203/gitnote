环境：我们使用之前部署的zookeeper集群，因为Kafka服务依赖于zookeeper服务。
三个节点： IP地址		主机名			节点
	192.168.200.169		zookeeper1		集群节点
	192.168.200.170		zookeeper2		集群节点
	192.168.200.171		zookeeper3		集群节点

所需软件包：kafka_2.11-1.1.1.tgz						//这里提供下载地址：http://kafka.apache.org/downloads.html

1：解压缩 Kafka 软件包（3个节点操作）
[root@zookeeper1 ~]# tar -zxvf kafka_2.11-1.1.1.gz			//解压到 /root下

2：修改3个基点配置文件（3个节点操作）
[root@zookeeper1 ~]# vi kafka_2.11-1.1.1/config/server.properties	//进入配置文件
#找到下列两行，并注释掉
broker.id=0
zookeeper.connect=localhost:2181					

#然后在配置文件的底部添加如下内容：					////注意，这里的listeners IP地址是 zookeeper1 的 IP地址
broker.id=1
zookeeper.connect=192.168.200.169:2181,192.168.200.170:2181,192.168.200.171:2181
listeners = PLAINTEXT://192.168.200.169:9092
注意：其他两个节点的配置都一样，但 broker.id 和 listeners 不能一样

3：启动服务（3个节点操作）
[root@zookeeper1 ~]# cd kafka_2.11-1.1.1/bin/				//进入目录
[root@zookeeper1 bin]# ./kafka-server-start.sh -daemon ../config/server.properties 	//启动服务
[root@zookeeper1 bin]# jps						//查看端口
1569 QuorumPeerMain
1938 Kafka
2007 Jps

4：测试服务
[root@zookeeper1 ~]# cd kafka_2.11-1.1.1/bin/				//进入目录
[root@zookeeper1 bin]# ./kafka-topics.sh --create --zookeeper 192.168.200.169 --replication-factor 1 --partitions 1 --topic test	//创建 topic
Created topic "test".	#这里成功才会显示

5：查看topic
#这里虽然 topic 是在 192.168.200.169 上创建的，但是在其他计算机上也能看到。
 例如，在任意启动的计算的 kafka_2.11-1.1.1/bin 目录中执行如下命令。
[root@zookeeper2 bin]# ./kafka-topics.sh --list --zookeeper 192.168.200.170:2181	#我们可以看到，在192.168.200.169上创建的 topic会话，同步到了其他服务器上
test
测试成功。