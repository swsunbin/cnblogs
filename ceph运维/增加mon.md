## 增加mon
集群初始状态
```bash
[root@node01 ~]# ceph -s
  cluster:
    id:     33af1a28-8923-4d40-af06-90c376ed74b0
    health: HEALTH_WARN
            Degraded data redundancy: 418/627 objects degraded (66.667%), 47 pgs degraded, 136 pgs undersized

  services:
    mon: 1 daemons, quorum node01 (age 23m)
    mgr: node01(active, since 19m)
    mds: cephfs:1 {0=node01=up:active}
    osd: 3 osds: 3 up (since 16m), 3 in (since 16m)
    rgw: 1 daemon active (node01)
 
  task status:
 
  data:
    pools:   6 pools, 136 pgs
    objects: 209 objects, 3.4 KiB
    usage:   3.0 GiB used, 57 GiB / 60 GiB avail
    pgs:     418/627 objects degraded (66.667%)
             89 active+undersized
             47 active+undersized+degraded
```
1. 关闭防火墙和selinux：
```bash
systemctl stop firewalld && systemctl disable firewalld
setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```
2.  分别在node02、node03创建目录：
```bash
sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node02
sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node03
```
3. 分别在node02、node03 执行命令如下：
```bash
scp root@192.168.19.101:/etc/ceph/ceph.client.admin.keyring /etc/ceph/
scp root@192.168.19.101:/etc/ceph/ceph.conf /etc/ceph/
```
4. node02、node03修改conf文件为如下：
```bash
[global]
auth client required = cephx
auth cluster required = cephx
auth service required = cephx
# auth supported = none
cluster network = 0.0.0.0/0
fsid = 33af1a28-8923-4d40-af06-90c376ed74b0
mon host = [v2:192.168.19.101:3300,v1:192.168.19.101:6789],[v2:192.168.19.102:3300,v1:192.168.19.102:6789],[v2:192.168.19.103:3300,v1:192.168.19.103:6789]
mon initial members = node01,node02,node03
mon_max_pg_per_osd = 600
mon_osd_down_out_interval = 8640000
osd pool default crush rule = -1
osd_pool_default_min_size = 1
osd_pool_default_size = 3
public network = 192.168.19.101/26
mon_allow_pool_delete = true
```
5. 在node01上执行以下命令获取mon 的keyring (需要读取本机的 `/etc/ceph/ceph.client.admin.keyring` 认证来获取mon的keyring):
```bash
sudo ceph auth get mon. -o /tmp/ceph.mon.keyring
```
6. 获取mon monmap:
```bash
sudo ceph mon getmap -o /tmp/monmap
```
查询内容输出如下：
```bash
[root@node01 tmp]# monmaptool --print /tmp/monmap
monmaptool: monmap file /tmp/monmap
epoch 1
fsid 33af1a28-8923-4d40-af06-90c376ed74b0
last_changed 2023-08-18 20:32:23.733383
created 2023-08-18 20:32:23.733383
min_mon_release 14 (nautilus)
0: [v2:192.168.19.101:3300/0,v1:192.168.19.101:6789/0] mon.node01
```
7. 但是需要注意，这里从集群中获得的 monmap 只包含了第一台服务器 node01 ，我们还需要添加增加节点node02、node03
```bash
monmaptool --addv node02 [v2:192.168.19.102:3300,v1:192.168.19.102:6789] --fsid 33af1a28-8923-4d40-af06-90c376ed74b0  /tmp/monmap
monmaptool --addv node03 [v2:192.168.19.103:3300,v1:192.168.19.103:6789] --fsid 33af1a28-8923-4d40-af06-90c376ed74b0  /tmp/monmap
```
查询内容输出：
```bash
[root@node01 ~]# monmaptool --print /tmp/monmap
monmaptool: monmap file /tmp/monmap
epoch 1
fsid 33af1a28-8923-4d40-af06-90c376ed74b0
last_changed 2023-08-18 20:32:23.733383
created 2023-08-18 20:32:23.733383
min_mon_release 14 (nautilus)
0: [v2:192.168.19.101:3300/0,v1:192.168.19.101:6789/0] mon.node01
1: [v2:192.168.19.102:3300/0,v1:192.168.19.102:6789/0] mon.node02
2: [v2:192.168.19.103:3300/0,v1:192.168.19.103:6789/0] mon.node03
```
现在`/tmp/monmap` 是最新的最全的monmap，但是在 node01上没有添加过`node02`和`node03`的`monmap`, 我们需要把这个 /tmp/monmap 插入到 node01 的监控目录中。所以把这个最新 /tmp/monmap 复制到 node01 上，再执行以下命令更新:

注意：更新`node01`的`monmap`之前，需要先停止`ceph-mon`否则会报错无法拿到db的锁。<br>
```bash
sudo systemctl stop ceph-mon@node01
``` 
```bash
sudo ceph-mon -i node01 --inject-monmap /tmp/monmap
```
8. 修改 /etc/ceph/cceph.conf 如上，启动 mon:
```bash
sudo systemctl start ceph-mon@node01
```
此时执行ceph -s无响应，启动node02、node03即可。

9. 在node02、node03分别执行如下命令：
```bash
scp root@192.168.19.101:/tmp/monmap /tmp/
scp root@192.168.19.101:/etc/ceph/ceph.client.admin.keyring /etc/ceph/
```
10. 在node02、node03分别执行如下：
```bash
sudo -u ceph ceph-mon --mkfs -i node02 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
sudo -u ceph ceph-mon --mkfs -i node03 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
```
11. 集群查询状态:
```bash
[root@node01 ~]# ceph -s
  cluster:
    id:     33af1a28-8923-4d40-af06-90c376ed74b0
    health: HEALTH_WARN
 
  services:
    mon: 3 daemons, quorum node01,node02,node03 (age 19m)
    mgr: node01(active, since 19m)
    mds: cephfs:1 {0=node01=up:active(laggy or crashed)}
    osd: 9 osds: 9 up (since 19m), 9 in (since 38h)
    rgw: 1 daemon active (node01)
 
  task status:
 
  data:
    pools:   7 pools, 140 pgs
    objects: 209 objects, 3.4 KiB
    usage:   9.6 GiB used, 170 GiB / 180 GiB avail
    pgs:     140 active+clean

 
```


