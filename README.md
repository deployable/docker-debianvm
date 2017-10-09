# Docker Debian VM

Vagrant Debian vm running the latest Docker, with a host accessible bridge network.

```
osx# vagrant up
osx# vagrant ssh
dkr$ docker run --net brocker0 --ip 192.168.98.21 debian ping 192.168.98.1
```

`eth0` is the standard NAT interface for Vagrant

`eth1` is a host only adapter attached to the 192.168.98.0/24 network

The `brocker0` bridge has eth1 attached and is assigned 192.168.98.30

