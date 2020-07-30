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

2：创建数据表
[root@localhost ~]# mysqladmin -uroot -p000000 create test	//创建一个名为 “test” 的数据库
[root@localhost ~]# mysql -uroot -p000000			//进入数据库
MariaDB [test]>	 use test;
MariaDB [test]>  CREATE TABLE IF NOT EXISTS `tables`(`tables_id` INT UNSIGNED AUTO_INCREMENT, `tables_title` VARCHAR(100) NOT NULL, `tables_author` VARCHAR(40) NOT NULL,`tables_data` DATE, PRIMARY KEY (`tables_id`)) ENGINE=INNODB DEFAULT CHARSET=utf8;

3：数据库备份
[root@localhost ~]# mysqldump -uroot -p000000 test > test.sql	//导出整个数据表
[root@localhost ~]# ls		#可以看到/root/目录下 有一个test.sql
anaconda-ks.cfg  CentOS-7-x86_64-DVD-2003.iso  PXE.sh  test.sql
#到处一个表，命令如下
[root@localhost ~]# mysqldump -uroot -p000000 test tables > test_tables.sql
[root@localhost ~]# ls
anaconda-ks.cfg  CentOS-7-x86_64-DVD-2003.iso  PXE.sh  test.sql  test_tables.sql

删除test数据库，进行导入测试，用 mysqldump 备份的文件是一个可以直接导入的SQL脚本。有两种方法可以将数据库导入，一种是 msql 命令，把数据库文件恢复到指定的数据库，命令如下：

