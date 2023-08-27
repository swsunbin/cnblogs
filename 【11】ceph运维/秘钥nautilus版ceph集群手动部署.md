## mon部署 

1. 为集群生成一个唯一的ID(即fsid)：

```
uuidgen
```

2. 为此集群创建密钥环 keyring 并生成监视器密钥 ：

```bash
sudo ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'
```

​	输出：

```bash
[root@node01 ~]# cat /tmp/ceph.mon.keyring
[mon.]
	key = AQBQYt9kQr8SDBAA05KJPJ7wj/cojuhu6rvVWQ==
	caps mon = "allow *"
```

生成管理员密钥环 keyring，生成 client.admin 用户并加入密钥环：

```bash
sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *'
```

输出：

```bash
[root@node01 ~]# cat /var/lib/ceph/bootstrap-osd/ceph.keyring
[client.bootstrap-osd]
	key = AQCmYt9kmTFMBBAAW8+Rwnv3iuRU6f2qVo5kXw==
	caps mgr = "allow r"
	caps mon = "profile bootstrap-osd"
```

将生成的keyring 导入到 ceph.mon.keyring：

```bash
sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
```

输出：

```b ash
[root@node01 ~]# sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
importing contents of /etc/ceph/ceph.client.admin.keyring into /tmp/ceph.mon.keyring
[root@node01 ~]# cat /tmp/ceph.mon.keyring
[mon.]
	key = AQBQYt9kQr8SDBAA05KJPJ7wj/cojuhu6rvVWQ==
	caps mon = "allow *"
[client.admin]
	key = AQB3Yt9kcV0CHxAA2ljATohTAe0u75zEtyTgLA==
	caps mds = "allow *"
	caps mgr = "allow *"
	caps mon = "allow *"
	caps osd = "allow *"
[root@node01 ~]# sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
importing contents of /var/lib/ceph/bootstrap-osd/ceph.keyring into /tmp/ceph.mon.keyring
[root@node01 ~]# cat /tmp/ceph.mon.keyring
[mon.]
	key = AQBQYt9kQr8SDBAA05KJPJ7wj/cojuhu6rvVWQ==
	caps mon = "allow *"
[client.admin]
	key = AQB3Yt9kcV0CHxAA2ljATohTAe0u75zEtyTgLA==
	caps mds = "allow *"
	caps mgr = "allow *"
	caps mon = "allow *"
	caps osd = "allow *"
[client.bootstrap-osd]
	key = AQCmYt9kmTFMBBAAW8+Rwnv3iuRU6f2qVo5kXw==
	caps mgr = "allow r"
	caps mon = "profile bootstrap-osd"
```

更改 ceph.mon.keyring 的所有者：

```bash
sudo chown ceph:ceph /tmp/ceph.mon.keyring
```

使用主机名、主机 IP 地址和 FSID 生成monitor映射，保存在 文件 /tmp/monmap中：

``` bash
monmaptool --create --addv node01 [v2:192.168.19.101:3300,v1:192.168.19.101:6789]  --fsid 33af1a28-8923-4d40-af06-90c376ed74b0  /tmp/monmap
```

在monitor主机上创建默认数据目录，目录名是{cluster-name}-{hostname}格式 :

```bash
sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node01
```

在node1节点对monitor进行初始化:

```bash
sudo -u ceph ceph-mon --mkfs -i node01  --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
```

将配置文件拷贝到其他节点：

```bash
scp /etc/ceph/ceph.conf node2:/etc/ceph/ceph.conf
scp /etc/ceph/ceph.conf node3:/etc/ceph/ceph.conf
```

 将mon keyring,mon map及admin keyring拷贝到其他节点

```bash
scp /tmp/ceph.mon.keyring node2:/tmp/ceph.mon.keyring
scp /etc/ceph/ceph.client.admin.keyring node2:/etc/ceph/ceph.client.admin.keyring
scp /tmp/monmap node2:/tmp/monmap

scp /tmp/ceph.mon.keyring node3:/tmp/ceph.mon.keyring
scp /etc/ceph/ceph.client.admin.keyring node3:/etc/ceph/ceph.client.admin.keyring
scp /tmp/monmap node3:/tmp/monmap
```



···

