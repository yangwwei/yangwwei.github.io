---
title: Docker Network Tests under Host/Bridge Mode
tags:
  - Docker
---

Recently we are considering to dockernize our computation slots and integrate them
into a Kubernetes like container management system. There are two network models
in docker, `Host` vs `Bridge`, and we have some debates on which one to adopt. This
post introduces some performance testing result to support our decision.

<!--more-->

If you don't know about docker network types, please see [this docker document](https://docs.docker.com/engine/userguide/networking/). I am testing
network performance with following scenarios.

1. Docker container with `Bridge` mode
2. Docker container with `Host` mode
3. Physical machines without docker container

To measure the network performance, I am leveraging a Linux tool [qperf](https://linux.die.net/man/1/qperf).
The tests are really simple, in each scenario, use qperf to measure the network
bandwidth and latency between two `hosts`, where the a host can be a docker container
or a physical machine.


```Shell
# install qperf
yum install qperf.x86_64

# On one host, start qperf server
qperf

# On the other host, run
qperf <host/ip> tcp_bw tcp_lat conf
```

Each case, I've run qperf for 5 times and counted an average value, see following result,

| Env | bandwidth(Bridge) | bandwidth(Host) | bandwidth(Physical) | latency(Bridge)  | latency(Host) |latency(Physical) |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Same physical machine | 4024 | 4639 | 4874 | 9.04 | 7.9 | 6.73 |
| Two different physical machines | 1010 | 1013 | 1035 | 20 | 17.5 | 17.1 |

Summary:

1. If containers are running on same physical machine, network bandwidth under `Host` mode is
**15%** bigger than `Bridge` mode, but **5%** slower than just physical machine. Regarding of latency,
it is **14%** faster than `Bridge` mode but **17%** slower than physical machine.
2. If containers are running on two different physical machine, network bandwidth under `Host` mode is
**0.3%** bigger than `Bridge` mode, but **2.2%** slower than just physical machine. Regarding of latency,
it is **14%** faster than `Bridge` mode but **2.3%** slower than physical machine.

Simply speaking, if running on same physical machine, `Host` mode is a lot better than `Bridge`;
however, if running on two different machines, `Host` mode is only slightly better, the performance
can both achieve nearly to what physical machine's limit.
