环境，双节点，主（master），从（slave） ，master IP地址：192.168.200.100   slave IP地址：192.168.200.151

######双节点操作
1：配置yum源

2：关闭防火墙

3：[root@localhost ~]# yum install -y bind-chroot bind-utils						//安装，配置BIND（两个节点）
     [root@localhost ~]# rpm -ql bind-chroot								//查询是否安装正确
     [root@localhost ~]# cd /var/named/chroot/								//进入bind-chroot目录

4：复制相关文件，准备bind-chroot环境
     [root@localhost chroot]# cp -R /usr/share/doc/bind-9.11.4/sample/etc/* /var/named/chroot/etc/
     [root@localhost chroot]# cp -R /usr/share/doc/bind-9.11.4/sample/var/* /var/named/chroot/var/

5：创建dynamic目录，将相关文件设置为可写
     [root@localhost chroot]# cd var/named/
     [root@localhost named]# chmod -R 777 /var/named/chroot/var/named/data/
     [root@localhost named]# mkdir dynamic								//这里注意，如果当下目录有dynamic，则输入命令会失败，需要去到 /var/named/chroot/var/named 下面创建dynamic
     [root@localhost named]# chmod 777 /var/named/chroot/var/named/dynamic/		

6：[root@localhost chroot]# cp /etc/named.conf  /var/named/chroot/etc/named.conf 			//将named.conf文件复制到bind-chroot目录中，如果进入目录发现与图片不符合，需要重新复制，也就是再输入以下这个命令，并且覆盖 yes

7：编辑named.conf文件，代码如下	#这里需要修改和添加的地方，我为大家列了出来 				//第7步尤其坑爹
修改：options下的——listen-on port 53 {localhost;}; 改为 listen-on port {any;};
          options下的——allow-query{###这里面放的啥我忘了；} 改为allow-query{any;};
          bindkeys-file "/etc/named.root.key";改为 bindkeys-file "/etc/named.iscdlv.key";
添加：zone "test.com" {
	type master;
	file "test.com.zon"
};

8：[root@localhost chroot]# chown named /var/named/chroot/etc/named.conf 				//设置named.conf文件的用户权限为named

9：创建转发域
[root@localhost chroot]# cp /var/named/named.localhost /var/named/chroot/var/named/test.com.zon		//复制模板文件named.localhost 到 test.com.zon ，这注意test.com.zon 要自己打 table不出来

编辑test.com.zone文件
$TTL 1D
@       IN SOA  test.com. admin.test.com. (

                                        2019001 ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H      ; minimum
)
        IN NS nsl.test.com.

nsl     IN A        192.168.200.200		#ip地址要修改成自己虚拟机网段的
www IN A     192.168.200.210
ftp    IN A        192.168.100.220

[root@localhost named]# chmod 777 test.com.zon							//赋予test.com.zone所有权限

10：检查配置
[root@localhost named]# named-checkconf /var/named/chroot/etc/named.conf 			//无返回结果则配置正确
[root@localhost named]# named-checkzone test.com test.com.zon 					//返回结果zone test.com/IN: loaded serial 2019001  OK

11：配置服务
[root@localhost named]# date -s 12:55:00							//配置时间#以自己的时间为准
[root@localhost named]# systemctl stop named							//关闭named服务
[root@localhost named]# systemctl disable named							//取消开机自启

[root@localhost named]# systemctl enable named-chroot 						//设置bind-chroot服务开机启动，并重启

root@localhost named]#ln -s '/usr/lib/systemd/system/named-chroot.service' '/etc/systemd/system/multi-user.target.wants/named-chroot.service'			//注意，如果输入这段命令出现下面的报错提示的话不要鸟它
报错提示——ln: failed to create symbolic link ‘/etc/systemd/system/multi-user.target.wants/named-chroot.service’: File exists

还有一种是[root@localhost ~]# systemctl enable named-chroot ln -s '/usr/lib/systemd/system/named-chroot.service' '/etc/systemd/system/multi-user.target.wants/named-chroot.service'  	//如果这样输入也报错的话也别鸟它。

[root@localhost ~]# cd /etc/systemd/system/multi-user.target.wants/ 	#只要保证在这个文件下，有named-chroot.service这个东西就行	

[root@localhost ~]# systemctl restart named-chroot 	//重启named-chroot服务
#如果报错，使用下面方法
[root@localhost ~]# vi /etc/sysconfig/named 		//进入这个配置文件
DISABLE_ZONE_CHECKING= "yes"		#在下面加上这句
然后重启服务即可

12：配置主机DNS服务器
[root@localhost ~]# vi /etc/resolv.conf     		//进入后添加或者修改  nameserver 192.168.200.100 ——这里的ip地址为自己的主机IP，（slave也用它自己的ip）

13：使用BIND基本命令重载主配置文件和区域解析库文件	#如果在输入第二个命令出错的话，自己去看以下自己named.conf的配置是不是有问题，然后重新来一遍命令rndc reload
[root@localhost named]# rndc reload
提示信息：server reload successful

[root@localhost named]# rndc reload
提示信息：server reload successful

[root@localhost named]# rndc reload test.com
提示信息：zone reload up-to-date

[root@localhost named]# rndc notify test.com
提示信息zone notify queued

[root@localhost named]# rndc reconfig
##然后测试DNS解析是否正常，这里选择ping以下DNS 地址，nslookup解析也可以，出现丢包也没关系，只要地址显示出来就ok
	DNS解析正常会显示下面消息
[root@localhost named]# ping www.test.com
PING www.test.com (192.169.200.120) 56(84) bytes of data.
64 bytes from ip-192-169-200-120.ip.secureserver.net (192.169.200.120): icmp_seq=1 ttl=128 time=173 ms
64 bytes from ip-192-169-200-120.ip.secureserver.net (192.169.200.120): icmp_seq=3 ttl=128 time=172 ms
64 bytes from ip-192-169-200-120.ip.secureserver.net (192.169.200.120): icmp_seq=4 ttl=128 time=172 ms
^C
--- www.test.com ping statistics ---
5 packets transmitted, 3 received, 40% packet loss, time 4027ms
rtt min/avg/max/mdev = 172.816/172.902/173.058/0.356 ms

14：配置主从DNS
####（master节点配置）
在master上面操作，修改master的named.conf文件
修改test.com，主要增加下面三项信息，allw-transfer，notify yes 和 also-notify	 #这里的IP地址是slave的IP地址
zone "test.com"{
        type master;
        file "test.com.zon";
        allow-transfer {192.168.200.151;};
        notify yes;
        also-notify {192.168.200.151;};
};

在master上编辑主服务器解析库文件 
vi /var/named/test.com.zon

$TTL 1D
$ORIGIN test.com.
@       IN SOA  test.com. admin.test.com. (
                                        2019002 ; serial       //值修改的比以前的大，才能同步
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H      ; minimum
)
                IN NS nsl.test.com.


nsl  IN A       192.168.200.110
www  IN A       192.168.200.120
www2 IN A       192.168.200.121 	//添加
ftp  IN A       192.168.200.130

重新加载配置文件 ##在master上操作
[root@localhost named]# rndc reload
[root@localhost named]# tail -f /var/log/messages

###在slave上操作，修改slave 服务器上的named.conf文件
修改部分，如下
zone "test.com" {
        type slave;
        file "slaves/test.com.zon";
        master {192.168.200.100;};
};

设置slaves目录权限和目录的所有者为named用户					//注意 一个是chmod，一个是chown
[root@localhost named]# chmod -R 777 /var/named/chroot/var/named/slaves/			
[root@localhost named]# chown -R named:named /var/named/chroot/var/named/slaves/ 

检查语法slave语法，并且在slave和master上重启服务
[root@localhost named]# named-checkconf /var/named/chroot/etc/named.conf 
[root@localhost named]# systemctl restart named-chroot

查看从服务器是否有文件同步进来
[root@localhost named]# ll /var/named/chroot/var/named/slaves/ 		#显示信息如下
total 12
-rwxrwxrwx. 1 named named   56 Jul 17 13:07 my.ddns.internal.zone.db
-rwxrwxrwx. 1 named named   56 Jul 17 13:07 my.slave.internal.zone.db
-rw -r --r --  1 named named 309 Jul 17 13:07 test.com.zon

在master上用从服务器解析（在@后面指定DNS服务器地址，就不用修改本机的NDS），解析到www2域名，表面配置成功

[root@localhost named]# dig www2.test.com @192.168.200.151  

到此linux 的DNS 服务与配置也就完成了，这玩意儿花了我4天去理解和配置，希望大家能从笔记中学到自己需要的东西，最后谢谢大家费心观看小生的笔记。
