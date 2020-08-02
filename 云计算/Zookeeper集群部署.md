环境：	IP地址			主机名			节点
	192.168.200.169		zookeeper1		集群节点
	192.168.200.170		zookeeper2		集群节点
	192.168.200.171		zookeeper3		集群节点

基础准备：使用提供的 zookeeper-3.4.14.tar.gz 包 和 gpmall-repo 文件夹安装 Zookeeper 服务
这里提供 zookeeper-3.4.14 压缩包下载地址：https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/

1：基础环境配置
#使用 sourceCRT 对3台linux主机进行连接。，将三个节点的主机名更改
	节点1：[root@localhost ~]# hostnamectl set-hostname zookeeper1
	节点2：[root@localhost ~]# hostnamectl set-hostname zookeeper2
	节点3：[root@localhost ~]# hostnamectl set-hostname zookeeper3

2：配置 hosts 文件,三个节点配置
	[root@zookeeper1 ~]# vi /etc/hosts
	192.168.200.169 zookeeper1
	192.168.200.170 zookeeper2
	192.168.200.171 zookeeper3

3：搭建 Zookeeper 集群

	（1）安装 JDK 环境		#在三个节点安装 JDK 环境
	[root@zookeeper1 ~]# yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel
  
	（2）解压缩包 zookeeper 软件包	#三个节点操作
	将压缩包，使用sourceCRT上传至/root目录下，并解压
	[root@zookeeper1 ~]# tar -zxvf zookeeper-3.4.14.tar.gz

	（3）修改3个节点配置文件
	[root@zookeeper1 conf]# mv zoo_sample.cfg zoo.cfg	//修改文件zoo_sample.cfg 名为 zoo.cfg
	#修改配置如下
	[root@zookeeper1 conf]# vi zoo.cfg			//进入配置文件
		tickTime=2000
		initLimit=10
		syncLimit=5
		dataDir=/tmp/zookeeper
		clientPort=2181
		server.1 = 192.168.200.169:2888:3888
		server.2 = 192.168.200.170:2888:3888
		server.3 = 192.168.200.171:2888:3888
	[root@zookeeper1 conf]# grep -n '^'[a-Z] zoo.cfg 	//我们输入这条命令，只要显示结果如下，就代表配置正确，最后三条前面的序号不用在意。
		2:tickTime=2000
		5:initLimit=10
		8:syncLimit=5
		12:dataDir=/tmp/zookeeper
		14:clientPort=2181
		15:server.1 = 192.168.200.169:2888:3888
		16:server.2 = 192.168.200.170:2888:3888
		17:server.3 = 192.168.200.171:2888:3888
	
	（4）创建 myid 文件
	#在三台linux dataDir 目录（此处为/tmp/zookeeper） 下分别创建一个 myid 文件，文件内容只有一行
	 分别是1、2、3。即文件中只有一个数字，这个数字即 zoo.cfg 配置文件中指定的值。zookeeper 根据该
	 文件来决定 zookeeper 集群中各计算机的身份分配
	
	#创建 myid 文件命令如下：
		zookeeper 节点1
		[root@zookeeper1 ~]# mkdir /tmp/zookeeper
		[root@zookeeper1 ~]# vi /tmp/zookeeper/myid
		[root@zookeeper1 ~]# cat /tmp/zookeeper/myid 
		1
		zookeeper 节点2
		[root@zookeeper1 ~]# mkdir /tmp/zookeeper
		[root@zookeeper1 ~]# vi /tmp/zookeeper/myid
		[root@zookeeper1 ~]# cat /tmp/zookeeper/myid 
		2
		zookeeper 节点3
		[root@zookeeper1 ~]# mkdir /tmp/zookeeper
		[root@zookeeper1 ~]# vi /tmp/zookeeper/myid
		[root@zookeeper1 ~]# cat /tmp/zookeeper/myid 
		3
	
		