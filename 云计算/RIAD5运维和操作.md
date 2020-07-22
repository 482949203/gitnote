环境：新建4个虚拟磁盘分区，每个2‎0GB大小，用3个20G的分区来模拟RAID5，第4个分区为热备盘

1：RAID5运维操作
	[root@localhost ~]# mdadm -Cv /dev/md5 -l5 -n3 /dev/sdb /dev/sdc /dev/sdd --spare-devices=1  /dev/sde
	[root@localhost ~]# mdadm -D /dev/md5      	 //显示md5的详细信息

2：模拟硬盘故障
	[root@localhost ~]# mdadm -f /dev/md5 /dev/sdb   //会显示这个#mdadm: set /dev/sdb faulty in /dev/md5
	[root@localhost ~]# mdadm -D /dev/md5 		 //显示md5的详细信息，可以看到有一个硬盘sdb显示faulty，但热备盘sde正在参与RADI5的重建

3：移除故障盘
	[root@localhost ~]# mdadm -r /dev/md5 /dev/sdb
	[root@localhost ~]# mdadm -D /dev/md5		 //查看md5信息

4：格式化RAID并进行挂载
	[root@localhost ~]# mkfs.xfs -f /dev/md5	 //-f命令，挂载时显示挂载信息
	[root@localhost ~]# mount /dev/md5 /mnt/	 //挂载时，可能会显示，mount: /dev/md5: can't read superblock
	解决方法如下： #这里提供若干方法，原因是linux版本不同
	1——fsck /dev/xxx 				//使用fsck修复硬盘
	2——xfs_repair /dev/md5				//使用xfs修复硬盘
	网上例子——xfs_repair /dev/sda1 |grep superblock
	网上例子——xfs_repair /dev/mapper/centos-home -L （根据自己的环境改变命令，要学会变通，有自己的思维）
	