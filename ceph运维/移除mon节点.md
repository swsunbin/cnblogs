## 删除mon节点
当要删除一个 mon 时，需要考虑删除后剩余的 mon 个数是否能够达到法定人数：
1. 停止 mon 进程：
```bash
systemctl stop ceph-mon@node01
```
2. 从集群中删除 mon：
```bash
ceph mon remove node01
```
3. 从 ceph.conf 中移除 mon 的入口部分（如果有）。