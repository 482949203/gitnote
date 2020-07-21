DHCP服务与管理笔记
环境：双节点 server IP 地址：192.168.200.158  client IP 地址：192.168.200.159

1：安装DHCP服务
[root@localhost ~]# yum install -y dhcp

2：修改网卡配置
[root@localhost ~]# cd /etc/sysconfig/network-scripts/
[root@localhost network-scripts]# ls
ifcfg-ens1677736  ifdown-ippp    ifdown-sit       ifup-bnep  ifup-plusb   ifup-TeamPort
ifcfg-ens33       ifdown-ipv6    ifdown-Team      ifup-eth   ifup-post    ifup-tunnel
ifcfg-lo          ifdown-isdn    ifdown-TeamPort  ifup-ippp  ifup-ppp     ifup-wireless
ifdown            ifdown-post    ifdown-tunnel    ifup-ipv6  ifup-routes  init.ipv6-global
ifdown-bnep       ifdown-ppp     ifup             ifup-isdn  ifup-sit     network-functions
ifdown-eth        ifdown-routes  ifup-aliases     ifup-plip  ifup-Team    network-functions-ipv6


ifcfg-ens33是本次实验网卡，我们vim 进入网卡
BOOTPROTO=static 注意，若将本机作为DHCP服务器，这里选择静态地址，client的BOOTPROTO=dhcp

3：修改DHCP配置文件，代码如下。![DHCP.png](1)

4：启动DHCP服务并设置开机自启，查看服务状态
[root@localhost ~]# systemctl start dhcpd
[root@localhost ~]# systemctl enable dhcpd 	//加入开机项目
[root@localhost ~]# systemctl start dhcpd