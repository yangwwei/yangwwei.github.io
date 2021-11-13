---
title: Spark Remote Shuffling Service
tags:
  - Spark
  - Shuffle
  - K8s
---
Explains a few buzz words in Spark, External Shuffle Service (ESS), Remote Shuffle Service (RSS).
And a few existing ESS, RSS solutions for cloud use cases.

<!--more-->

## External Shuffle Service on YARN

Spark has a built-in External Shuffle Service (ESS) on YARN, it runs as a long-running
auxiliary on each node manager. When `spark.shuffle.service.enabled` is enabled,
Spark executors will register with the ESS and connect with ESS via shuffle client.
The shuffle blocks will be preserved in YARN node manager local dir, so the data
is prevserved even the executor is idle and removed by the driver. ESS is required to be
enabled before using [dynamica allocation](https://spark.apache.org/docs/latest/job-scheduling.html#dynamic-resource-allocation).

## Push-based Shuffle Service (Magnet)

LinkedIn has published a paper [Magnet: Push-based Shuffle Service for Large-scale Data Processing](http://www.vldb.org/pvldb/vol13/p3382-shen.pdf)
to explain the optimizations on top of ESS. With its so called push-merge style, it saves disk I/O by combining
smaller chunks of shuffle blocks to larger ones, and it tolerates failures as it only does the best-effort push.
This increase the reliability of the shuffle service, as a result, Spark job runs are more stable and possibly achived better performance.
Magnet now is available in [Spark 3.2](https://spark.apache.org/releases/spark-release-3-2-0.html) via [SPARK-30602](https://issues.apache.org/jira/browse/SPARK-30602). And one thing to notice is, Magnet stores the shuffle data mapping info directly in Spark driver,
this avoids to having a single-point contact (usually a server role in the RSS) to store such info, which could potentially become a bottleneck when it scales out.

## The problem of ESS on YARN

On-prem clusters, it is common to use Spark dynamical allocation with ESS. This saves
the resources because Spark can dynamically scale down the executors based on the needs,
this saves resources and can be used by other executors. ESS works very well when the cluster
size is mostly static, you don't need to scale up/down the cluster size very often, so the
shuffle data could be preserved on node managers to serve other executors. However, when
it comes to the cloud,the assumption no longer stands.

On cloud, the pricing mode is pay-as-you-go. Running large scale
computations on cloud requires to use resources efficiently, this requires to dynamically scale
the cluster based on the actual needs. In this case, there will be no long-running nodes available for running
ESS instances. If the cluster is being dynamically resized frequently, it will cause a lot of
overhead to recompute task stages as the shuffle data are lost along with the decommissioned nodes.
This is why [disaggregated compute and storage architecture](https://en.wikipedia.org/wiki/Disaggregated_storage) is important on cloud.

Some of the existing practices is to continue use YARN ESS on cloud for Spark, this works but less efficiently.
It is a anti-pattern to the cloud while requires to have long-running nodes. To achieve better efficiency and reliability,
there are many Remote Shuffle Service (RSS) being developed.

## Remote Shuffle Service

The remote shuffle service takes a further step by storing the shuffle data in
remote storage. This is an output of disaggregated compute and storage architecture. On the mapper side,
it needs to push data to the remote RSS server before ending a stage, on the reducer side,
it needs to fetch data from remote RSS servers instead of local storage. With this disaggregated architecture,
the compuate nodes can be highly dynamical. With Spark DA enabled, the compute nodes can be scaled up and down
independently with the RSS nodes.

There are several RSS existing today, here is a list with a short description about their characteristic:

- [Zeus](https://github.com/uber/RemoteShuffleService): Developed by Uber. It is pushed based, executors push the shuffle data to the remote servers, and it can even write an extra replica to improve the reliability. Zeus doesn't have any master roles, all RSS instances simply accept shuffle blocks and store the data locally. It uses hash-based mechanism to route shuffle data to certain RSS node. Zeus is open sourced under Apache 2 license. Desing doc can be found [here](https://github.com/uber/RemoteShuffleService/blob/master/docs/server-high-level-design.md).
- [Alibaba EMR RSS](https://www.alibabacloud.com/blog/emr-remote-shuffle-service-a-powerful-elastic-tool-of-serverless-spark_597728): Developed by Alibaba EMR, push-based style. It has master, worker roles. Master coordinates the resource allocation, and the workers store and manage the shuffle data. It also supports to push an extra copy of shuffle block to improve the reliability. This is a proprietary software.
- [Tecent Firstorm](https://github.com/Tencent/Firestorm): Developed by Tecent, recently open sourced under Apache 2 license. Very similar to Alibaba EMR's solution.

## Remote Shuffle Service on K8s

As it for now, there is no open source solution available for providing RSS on K8s.
It is easy to setup and deploy RSS servers on K8s with K8s primitives. The key challenage
is how to provide a reliable, cost saving solution. Key problems like:
- Optimize storage and network for RSS
- Auto-scaling of RSS instances
- Fault tolerance
- Scalability

We will cover these topics in the future.
