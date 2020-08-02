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
	
