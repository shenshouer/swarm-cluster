# swarm-cluster

## require

* vagrant
* virtualbox
* coreos box

## architecture

* swarm-01 172.18.19.101 consul && swarm master
* swarm-02 172.18.19.102 swarm master
* swarm-03 172.18.19.103 swarm node
* swarm-04 172.18.19.104 swarm node


## start

```
git clone https://github.com/shenshouer/swarm-cluster.git && cd swarm-cluster

vagrant up
```

## cluster info
```
vagrant ssh swarm-01

core@swarm-01 ~ $ docker -H :4000 info
Containers: 3
 Running: 2
 Paused: 0
 Stopped: 1
Images: 3
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 swarm-03: 172.18.19.103:2375
  └ Status: Healthy
  └ Containers: 2
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 2.056 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.4.1-coreos, operatingsystem=CoreOS 955.0.0, storagedriver=overlay
  └ Error: (none)
  └ UpdatedAt: 2016-02-16T08:28:45Z
 swarm-04: 172.18.19.104:2375
  └ Status: Healthy
  └ Containers: 1
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 2.056 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.4.1-coreos, operatingsystem=CoreOS 955.0.0, storagedriver=overlay
  └ Error: (none)
  └ UpdatedAt: 2016-02-16T08:28:48Z
Plugins:
 Volume:
 Network:
Kernel Version: 4.4.1-coreos
Operating System: linux
Architecture: amd64
CPUs: 4
Total Memory: 4.112 GiB
Name: 95fbfa34d799
```

## check health

```
core@swarm-01 ~ $ docker -H :4000 run hello-world
core@swarm-01 ~ $ docker -H :4000 ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
core@swarm-01 ~ $ docker -H :4000 ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                      PORTS                                    NAMES
7db89ac5be53        hello-world         "/hello"                 24 seconds ago       Exited (0) 22 seconds ago                                            swarm-03/loving_mccarthy
8e9f17b678bc        swarm               "/swarm join --advert"   54 seconds ago       Up 52 seconds               2375/tcp, 172.18.19.104:4000->4000/tcp   swarm-04/swarm-node
9a36b3c81f62        swarm               "/swarm join --advert"   About a minute ago   Up About a minute           2375/tcp, 172.18.19.103:4000->4000/tcp   swarm-03/swarm-node
```

