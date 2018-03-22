# Docker Debian VM

Vagrant Debian VM running the latest Docker, with a host accessible bridge network for containers.

```
osx# vagrant up
osx# vagrant ssh

dkr$ sudo docker network inspect vmhost
dkr# CID=$(sudo docker run --net=vmhost --ip 192.168.98.20 -d --rm alpine sleep 1800)
dkr# sudo docker exec $CID ip ad sh eth0
dkr# sudo docker exec $CID ping -c 2 192.168.98.30   # Container -> Docker Host
dkr# sudo docker exec $CID ping -c 2 192.168.98.1    # Container -> VM Host

osx# ping -c 2 192.168.98.20                         # VM host -> container
PING 192.168.98.20 (192.168.98.20): 56 data bytes
64 bytes from 192.168.98.20: icmp_seq=0 ttl=64 time=0.479 ms
64 bytes from 192.168.98.20: icmp_seq=1 ttl=64 time=0.383 ms
```

### VM Interfaces

`eth0` is the standard NAT interface for Vagrant

`eth1` is a host only adapter attached to the 192.168.98.0/24 network

The `brocker0` bridge has eth1 attached and is assigned 192.168.98.30

Containers attached to the `vmhost` network on `brocker0` and assigned an address in the 
192.168.98.0/24 range become directly accessible from the VM Host. 


