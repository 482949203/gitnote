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
[root@localhost ~]# mysqladmin -uroot -p000000 create test	//创建一个名为 “test” 的数据库
[root@localhost ~]# mysql -uroot -p000000			//进入数据库

MariaDB [test]> CREATE TABLE IF NOT EXISTS 'tables'('tables_id' INT UNSIGNED AUTO_INCREMENT, 'tables_title' VARCHAR(100) NOT NULL, 'tables_author' VARCHAR(40) NOT NULL, 'tables_data' DATE, PRIMARY KEY ('tables_id')) ENGINE=InnoDB DEFAUTL CHARSET=utf8;

CREATE TABLE IF NOT EXISTS `Student2`(
`id` INT(4) NOT NULL AUTO_INCREMENT COMMENT '学号',
`name` VARCHAR(30) NOT NULL DEFAULT '匿名' COMMENT'姓名',
`pwd` VARCHAR(20) NOT NULL DEFAULT'123456' COMMENT'密码' ,
`sex` VARCHAR(2) NOT NULL DEFAULT '男'COMMENT'性别',
`birthday` DATETIME DEFAULT NULL COMMENT '出生日期',
`address`VARCHAR (100)DEFAULT NULL COMMENT '家庭住址',
`email`VARCHAR(50)DEFAULT NULL COMMENT '邮箱',