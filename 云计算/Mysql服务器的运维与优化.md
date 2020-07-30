环境：	单节点操作 mysql IP地址：192.168.200.165

1：安装环境
[root@localhost ~]# yum install -y mariadb mariadb-server	//安装Mariadb数据库
[root@localhost ~]# systemctl start mariadb			//开启数据库，否则后面可能会初始化失败
[root@localhost ~]# mysql_secure_installation			//初始化数据库
#初始化操作如下：
	Set root password? [Y/n] y
	Remove anonymous users? [Y/n] y
	Disallow root login remotely? [Y/n] n
	Remove test database and access to it? [Y/n] y
	Reload privilege tables now? [Y/n] y

2：配置数据库