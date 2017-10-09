# Docker Debian VM

Vagrant a debian vm running latest docker

```
osx# vagrant up
osx# vagrant ssh
dkr$ docker run --net brocker0 --ip 192.168.98.21 debian ping 192.168.98.1
```

eth0 standard NAT interface for Vagrant

eth1 host only adapter 192.168.98.30

brosx0 adapter 

eth2 host only adapter on brosx0

