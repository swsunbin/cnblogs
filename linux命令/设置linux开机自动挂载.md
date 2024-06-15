## 设置linux开机自动挂载

在/etc/rc.local内添加挂载命令，采用/etc/rc.local内添加挂载命令，如果输入命令错误，云服务器重启时不会影响操作系统正常运行。

> 注：该方法通过盘符进行自动挂载，云硬盘进行挂载卸载操作、云服务器硬重启时盘符会产生改变或者漂移，建议只有一块数据盘（vdb）时采用该方法设置自动挂载。

操作步骤

1. 打开`vi /etc/rc.local`文件，配置开机自动挂载，如下图所示：

   ![cnblogs](https://raw.githubusercontent.com/swsunbin/images/main/linux命令/FD0A2E45EB663665F02D581F4F76F09B.png))



1. 运行命令` cp /etc/fstab /etc/fstab.bak`，备份`etc/fstab`
2. 运行命令`blkid`查看文件系统的UUID，复制需要设置开机挂载的文件系统UUID及文件系统类型。这里`/dev/vdb1`的UUID为468f89f6-32b7-432f-bd98-34d6fd8ad375，文件系统类型为ext4

​	![cnblogs](https://raw.githubusercontent.com/swsunbin/images/main/linux命令/293AEFF0FF039B7586964E009F8D8A2E.png)



```c++
[root@ruo8h2pmn5wly9 ~]# cp /etc/fstab /etc/fstab.bak
[root@ruo8h2pmn5wly9 ~]# blkid /dev/vdb1
/dev/vdb1: UUID="468f89f6-32b7-432f-bd98-34d6fd8ad375" TYPE="ext4" PARTUUID="10b911a3-01"
[root@ruo8h2pmn5wly9 ~]# echo UUID=468f89f6-32b7-432f-bd98-34d6fd8ad375 /data ext4 defaults 0 0 >> /etc/fstab
[root@ruo8h2pmn5wly9 ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Wed Dec 25 06:58:40 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=f12446c3-9101-4256-b900-6b0963a0b0e0 /boot                   xfs     defaults        0 0
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/mapper/centos-swap	none	swap	sw,comment=cloudconfig	0	0
UUID=468f89f6-32b7-432f-bd98-34d6fd8ad375 /data ext4 defaults 0 0
```





## 参考资料

* [设置linux开机自动挂载](https://www.wangsu.com/document/878/evs-quickstart-format-file-system-linux-auto-mount)

  
