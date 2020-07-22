服务解析：
	若要为用户提供指定testmail域的电子邮件系统，需要现在DNS服务器中增加A记录和MX记录
	邮件传输涉及的组件
		MUA（Mail Transfer Agent）邮件用户代理人	功能：收取邮件主机的电子邮件，以及提供用户浏览和编写邮件
		MTA（Mail Transfer Agent )  邮件发送代理人	功能：受用简单邮件传输协议SMTP转发邮件
		MDA（Mail Delivery Agent）邮件传送代理人	功能：分析由MTA收到的邮件表头或能容等数据，以决定这封邮件的去向
	邮件的应用协议
		简单邮件传输协议（SMTP）：用来发送或转发出的电子邮件，占用TCP25端口
		第三版邮件协议（POP）：用于吧服务器上的邮件存储到本地主机，占用TCP110端口
		第四版互联网信息访问协议（IMAP4）：用于在本地主机山访问邮件，占用TCP143端口
	邮件系统构架
		基于SMTP的postfix服务提供发件服务功能，并使用POP3的dovecot服务程序提供收件服务功能、
	主要搭建步骤如下
		1.安装dovecot服务程序；
		2.修改dovecot程序主配置文件；
		3.配置邮件的格式域存储路径；
		4.创建邮件的存储目录；
		5.启动dovecot服务程序；

实战案例——E-Mail服务域管理笔记：
	环境：双节点规划，节点Mail服务器 IP地址：192.168.200.155 	节点DNS服务器，IP地址：192.168.200.156
双节点关闭防火墙：[root@localhost named]# setenforce 0	[root@localhost named]# systemctl stop firewalld

1：基础配置——Mail节点
[root@mail ~]# hostnamectl set-hostname mail.testmail.com	//修改主机名，这里不要怕麻烦，主机名可能在后面会用到

2：DNS服务配置——DNS节点
[root@localhost ~]# yum install -y bind-chroot bind-untils 	//安装服务
[root@localhost ~]# vi /etc/named.conf  			//修改DNS节点BIND配置文件named.conf
##修改和添加

修改：listen-on port 53 { 127.0.0.1; };——改为listen-on port 53 { any; };
          allow-query     { localhost; };——改为allow-query     { any; };
         bindkeys-file "/etc/named.root.key";——改为bindkeys-file "/etc/named.iscdlv.key";
添加：//添加代码实现正反向解析
zone "testmail.com" IN {
        type master;
        file "testmail.com.zone";
};

zone "200.168.192.in-addr.arpa"  IN {
//主机网段倒序写，实验网段是192.168.200.0
        type master;
        file "testmail.com.local";
};

[root@localhost ~]# cd /var/named/	//进入目录
[root@localhost named]# cp named.localhost testmail.com.zone	//复制named.localhost文件，并创建 mailtest.com.zone
[root@localhost named]# cp named.localhost testmail.com.local	//上同创建 mailtest.com.local

修改 mailtest.com.zone 文件 #修改结果如下

$TTL 86400
@       IN SOA  ns.testmail.com. admin.testmail.com. (
                                        2019008 ; serial
                                        2H      ; refresh
                                        10M     ; retry
                                        3D      ; expire
                                        1D      ; minimum
)
        IN NS nsl
        IN    MX 10     mail    
nsl     IN A    192.168.200.155			//这里IP地址是mail节点的IP地址
mail    IN A    192.168.200.155	

修改 testmail.com.local 文件  #修改结果如下

$TTL 86400
@       IN SOA  ns.testmail.com. admin.testmail.com. (
                                        2019003 ; serial
                                        2H      ; refresh
                                        10M     ; retry
                                        3D      ; expire
                                        1D      ; minimum
)
        
        IN NS   ns.testmail.com.
ns     A 	    192.168.200.155		
1       IN PTR  ns.testmail.com.
1       IN PTR  mail.testmail.com.                

[root@localhost named]# named-checkzone testmail.com testmail.com.zone	//检查配置文件是否有语法错误
[root@localhost named]# named-checkzone testmail.com testmail.com.local
[root@localhost named]# systemctl restart named-chroot	//重启服务

3：将Mail主机DNS解析指向DNS服务器——Mali节点操作

[root@mail ~]# vi /etc/resolv.conf		//进入配置文件
nameserver 192.168.200.156			//添加或修改为自己DNS服务器的IP地址，DNS地址为mail-master的IP地址
[root@mail ~]# yum install -y bind-utils	//安装dig工具
[root@mail ~]# dig -t A mail.testmail.com	//解析域名
测试结果如下：（图片）
[root@mail ~]# rpm -e postfix			//清除postfix安装包
[root@mail ~]# userdel postfix			//清除postfix用户
[root@mail ~]# groupdel postdrop		//清除postdrop用户组

#新建用户
[root@mail ~]# groupadd -g 2525 postfix
[root@mail ~]# useradd -g 2525 -u 2525 -M -s /sbin/nologin postfix
[root@mail ~]# groupadd -g 2526 postdrop
[root@mail ~]# useradd -g 2526 -u 2526 -M -s /sbin/nologin postdrop

[root@mail ~]# yum install -y postfix	//安装postfix  	##如果安装不起来检查一下yum源，和是否挂载到指定目录

配置postfix
[root@mail ~]# vim /etc/postfix/main.cf
 	
#修改邮局主机名					myhostname = mail.zhongdianjizhi.com
#修改邮局域名					mydomain = zhongdianjizhi.com
#寄出邮件域名，删除注释				myorigin = $mydomain
#修改监听所有网卡，删除注释			inet_interfaces = all
#修改可接邮件的主机名和域名、可被中继得到域名	mydestination = $myhostnamed,localhost,$mydoamin,localhost,$mydomian
#修改可接收邮件的主机名和域名，可被中继的主机	mynetworks = 192.168.118.0/24, 127.0.0.0/8
#取消注释					home_mailbox = Maildir/
#指定信任网段类型				mynetworks_style = host
#指定允许中转邮件的域名，取消注释			realy_domains = $mydestination

添加权限，设置开机启动并重启服务
[root@mail ~]# chown postfix.postfix -R /var/lib/postfix/
[root@mail ~]# chown postfix.postfix /var/spool/ -R
[root@mail ~]# systemctl enable postfix
[root@mail ~]# systemctl restart postfix
##重启不成功，输入下面命令
[root@mail ~]# vim /etc/postfix/main.cf
查看这里是不是都是ALL	——别问为什么两个一样的还要改，因为坑都被我踩遍了
inet_interfaces = all
inet_protocols = all
inet_interfaces = all

测试发送邮件
[root@mail ~]# useradd cwl				//创建测试邮件接收用户
[root@mail ~]# echo "111111" | passwd --stdin cwl	//发送邮件，并设置密码
##若出现下面消息，则发送成功。
Changing password for user cwl.
passwd: all authentication tokens updated successfully.

[root@mail ~]# yum install -y telnet			//安装telnet服务

###使用telnet命令链接邮件服务器25端口，发送邮件，具体测试如下，注意输入了telnet.mail.testmail.com 25 这个命令之后会停在那，那是在等你编辑邮件，我输入的地方我会用符号@提示
[root@mail ~]# telnet mail.testmail.com 25
Trying 192.168.200.155...
Connected to mail.testmail.com.
Escape character is '^]'.
220 mail.zhongdianjizhi.com ESMTP Postfix
@ mail from:root@testmail.com    			//发件人
250 2.1.0 Ok
@rcpt to:cw						//收件人				
250 2.1.5 Ok
@data							//填写邮件
354 End data with <CR><LF>.<CR><LF>
@hello,this is test mail.  				//输入邮件内容
@.							//以 “.”结束输入
250 2.0.0 Ok: queued as 7BFC9607DFFD
@quit							//退出
221 2.0.0 Bye
Connection closed by foreign host.

##查看发送状态
[root@mail ~]# tail /var/log/maillog | grep sent
Jul 19 09:58:13 mail postfix/local[2600]: 7BFC9607DFFD: to=<cwl@zhongdianjizhi.com>, orig_to=<cwl>, relay=local, delay=66, delays=66/0.03/0/0, dsn=2.0.0, status=sent (delivered to maildir)  	##这里status=sent，表示发送成功！

4：安装与配置dovecot
[root@mail ~]# yum install -y dovecot	//安装dovecot

修改dovecot相关配置文件，代码如下

[root@mail ~]# vim /etc/dovecot/dovecot.conf
#如果不使用IPv6,修改为 * 
listen = *

[root@mail ~]# vim /etc/dovecot/conf.d/10-auth.conf 	###注意，这里的第多少行，表示那个配置在那附近，而不是就一定在那，下面的也如此
#9行：取消注释并修改
#是否允许在没有SSL/TLS的情况下以明码登录
disbale_plaintext_auth = no 
#97行：添加
auth_mechanisms = plain login

[root@mail ~]# vim /etc/dovecot/conf.d/10-mail.conf 
#30行：取消注释并添加
mail_localtion = maildir:~/Maildir

[root@mail ~]# vim /etc/dovecot/conf.d/10-master.conf 
#88-99行：取消注释并添加postfix SMTP验证
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }

[root@mail ~]# systemctl restart dovecot		//重启服务

测试接收邮件，代码如下
[root@mail ~]# telnet mail.testmail.com 110	##这里也需要自己输入，不要在那傻等，我输入的地方会用@符号标注

Trying 192.168.200.155...
Connected to mail.testmail.com.
Escape character is '^]'.
+OK Dovecot ready.
@user cwl  				//登录用户
+OK
@pass 111111				//输入密码
+OK Logged in.
@List    				//邮件列表，可以看到我们之前发出的一个邮件，已经接收到了，+OK 1 messages
+OK 1 messages:
1 445
.
@retr 1					//输入邮件编号查看邮件
+OK 445 octets
Return-Path: <root@testmail.com>
X-Original-To: cwl
Delivered-To: cwl@zhongdianjizhi.com
Received: from mail.testmail.com (mail.testmail.com [192.168.200.155])
        by mail.zhongdianjizhi.com (Postfix) with SMTP id 7BFC9607DFFD
        for <cwl>; Sun, 19 Jul 2020 09:57:07 +0800 (CST)
Message-Id: <20200719015744.7BFC9607DFFD@mail.zhongdianjizhi.com>
Date: Sun, 19 Jul 2020 09:57:07 +0800 (CST)
From: root@testmail.com

hello,this is test mail.
.
@quit					//退出
+OK Logging out.
Connection closed by foreign host.

##到此，linux的E-mail服务与管理也就结束了，十分感谢大家观看我的笔记，希望对大家有所收获，笔记若有误欢迎大家指出！