环境：	单节点操作 mysql IP地址：192.168.200.165

1：安装环境
[root@localhost ~]# yum install -y mariadb mariadb-server	//安装Mariadb数据库
[root@localhost ~]# systemctl start mariadb			//开启数据库，否则后面可能会初始化失败
