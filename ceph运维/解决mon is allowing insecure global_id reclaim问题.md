# 解决mon is allowing insecure global_id reclaim问题
查询ceph状态：
```bash
$ ceph -s
  cluster:
    id:     37ac4cbb-a2c6-4f81-af1e-e9e39c010c85
    health: HEALTH_WARN
            mon is allowing insecure global_id reclaim
 
  services:
    mon: 1 daemons, quorum ceph-node-11 (age 64s)
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:   
```
解决方法：禁用不安全模式
```bash
ceph config set mon auth_allow_insecure_global_id_reclaim false
```
例如：
```bash
$ ceph config set mon auth_allow_insecure_global_id_reclaim false
$ ceph -s
  cluster:
    id:     37ac4cbb-a2c6-4f81-af1e-e9e39c010c85
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum ceph-node-11 (age 90s)
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:   
```