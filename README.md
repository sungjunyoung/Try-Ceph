![status-badge](https://img.shields.io/badge/status-WIP-yellow)

# Ceph Local
> Forked from https://github.com/ksingh7/Try-Ceph

## Requirements
- vagrant
- [vagrant-hosts](https://github.com/oscar-stack/vagrant-hosts) plugin
- ansible

## Getting start
```bash
$ vagrant up
... # let's take some coffee

$ vagrant ssh admin
[vagrant@admin ~]$ sudo ceph -s
  cluster:
    id:     fe00ca74-dc88-4a80-bf68-e30064151893
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum mon0 (age 36m)
    mgr: mgr0(active, since 28m)
    osd: 6 osds: 6 up (since 9m), 6 in (since 9m)
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   6.0 GiB used, 54 GiB / 60 GiB avail
    pgs:     

```

## Toolbox
- https://github.com/sungjunyoung/ceph-local/tree/master/toolbox