环境：添加4个20G的虚拟硬盘

RAID 0的运维操作

1:配置yum源文件 (如果有mdadm_yum这个文件就可以上传使用，若没有也没关系，不影响下面操作)

[mdadm]
name=mdadm
baseurl=file:///opt/mdadm_yum
gpgcheck=0
enabled=1

2: yum install -y mdadm    //安装mdadm

3：创建一个RAID0设备，这里使用/dev/sdb  /dev/sdc 做实验
#利用/dev/sdb 和/dev/sdc 建立等级为RAID 0 的 md0 （设备名）

mdadm -C -v /dev/md0 -l -0 -n 2 /dev/sdb /dev/sdc
命令解析： -C -v  //创建设备，并显示信息
 	 -l 0    //创建等级为 RAID 0
	 -n 2   //创建RAID 的设备为两个
cat /proc/mdstat 	//查看系统上的RAID
mdadm -Ds	//查看详细信息

4:mdadm -Ds > /etc/mdadm.conf 	//生成配置文件mdadm.conf

5:对创建的RAID进行文件系统创建（格式化mkfs.xfs）并挂载
mkfs.xfs /dev/md0
mkdir /raid0
mount /dev/md0 /raid0
df -Th /raid0/			//查看

6:设置为开机自动挂载
blkid /dev/md0 ————输入该命令后，会显示/dev/md0  的UUID 将其复制
echo "UUID=…………………… /raid0 xfs default 0 0 " >> /etc/fstab

7:删除RAID操作如下

umount /raid0
mdadm -S /dev/md0
rm -rf /etc/mdadm.conf
rm -rf /raid0
mdadm --zero-superblock /dev/sdb
mdadm --zero-superblock /dev/sdc
vi /etc/fstab ————进去后将开机自启的UUID 删除即可