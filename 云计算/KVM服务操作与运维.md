境：单节点规划，将cirros-0.3.3-x86_64-disk.img镜像与qemu-ifup-NAT脚本文件上传到root目录下    #由于我没有0.3.3所以用0.3.4代替，大家可以使用IDM下载器去官网下载0.3.3版本
小生在这里提供qemu-ifup-NAT脚本源码，也不劳费大家去github上找了


1：grep -E '(svm|vmx)' /proc/cpuinfo						//查看CPU是否支持虚拟化（这一步大家不必在乎返回结果）
#若物理CPU支持虚拟化，需要在虚拟机设置：处理器虚拟化Intel ，且需要在BIOS中开启VT，如果不支持也没关系，不影响下面操作，只是开不了KVM创建的虚拟机而已
#这里提供虚拟机设置截图

2：配置yum源

3：[root@localhost ~]# yum install -y qemu-kvm openssl libvirt			//使用yum命令安装KVM的主要组件及工具

4：[root@localhost ~]# systemctl start libvirtd					//启动libvirtd服务

5：[root@localhost ~]# ln -s /usr/libexec/qemu-kvm /usr/bin/qemu-kvm		//将usr/libexec/qemu-kvm链接为usr/bin/qemu-kvm

6：创建NAT模式KVM虚拟机
[root@localhost ~]# chmod 777 /root/qume-ifup-NAT 				//给予脚本执行权限（很多东西都可能是由于权限从而导致无法启动或使用的）

7：[root@localhost ~]# qemu-kvm -m 1024 file=/root/cirros-0.3.4-x86_64-disk.img,if=virtio -net nic,model=virtio -net tap,script=/root/qemu-ifup-NAT -nographic -vnc:1									//通过qemu-kvm命令启动KVM虚拟机——#注意自己的路径和镜像版本

8：验证
#创建虚拟机完成后，cirros用户登录虚拟机，输入用户名为cirros，密码为cubswin:)。然后输入ip a命令查询IP地址，最后输入route -n命令查询路由表即可完成
需要注意，若物理CPU支持VT虚拟化，则做在第7步创建启动过程即可，使用ip a 查看是否有这个IP地址即可——192.168.122.1


