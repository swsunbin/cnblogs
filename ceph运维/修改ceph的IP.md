## 修改ceph的IP
 
1. 导出`mon`配置

```bash
ceph mon getmap -o monmap.bin
```
2. 修改`mon`配置

```bash

#打印查看原来的mon配置
monmaptool --print monmap.bin
#删除原来mon配置(多个节点直接在后面加"--rm xxxx"即可)
monmaptool --rm node1 monmap.bin
#添加新的mon配置(多个就多家几个"--add nodeX xxxxx"即可)
monmaptool --add node1 192.168.17.15:6789 monmap.bin
#修改后打印一下,验证是否修改正确
monmaptool --print monmap.bin
```
3. 修改ceph的配置文件：将 `/etc/ceph.conf` 配置文件中原来的`IP`地址替换为新的ip地址(每一台都必须修改)。如果是用`ceph-deploy`安装的，可将安装目录下的`ceph.conf`修改掉，然后将配置文件同步到其他节点，如果ceph-deploy命令已经失效，那么就手动把配置文件拷贝到其他节点。

```bash
ceph-deploy --overwrite-conf admin node1
```
4. 关闭ceph集群

```bash
systemctl stop ceph-mon@node01
```
5. 修改服务器IP: 修改`/etc/sysconfig/network-scripts/ifcfg-XXXX`，将原来`/etc/hosts`内的域名配置中原来的IP进行替换，重启网卡：
```bash
systemctl restart network
```
6. 导入修改的mon:
```bash
ceph-mon -i node1  --inject-monmap monmap.bin
```

 
