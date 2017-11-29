---
title: Opportunistic Containers in Practice
tags:
  - YARN
---

This post is about some practices around opportunistic containers in yarn.

<!--more-->

### Enable Opportunistic Container

Set up a Hadoop cluster, and enable this feature with following configuration. More details in [apache doc](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/OpportunisticContainers.html)

```
<property>
  <name>yarn.resourcemanager.opportunistic-container-allocation.enabled</name>
  <value>true</value>
</property>

<property>
  <name>yarn.nodemanager.opportunistic-containers-max-queue-length</name>
  <description>The size of the queue for opportunistic containers in node manager</description>
  <value>3</value>
</property>
```
Save changes and restart yarn cluster.

### Run a test MR job

Run following command

```
/bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.0-SNAPSHOT.jar pi  -Dmapreduce.job.num-opportunistic-maps-percent="40" 50 100
```

By specifying `-Dmapreduce.job.num-opportunistic-maps-percent="40"`, we are expecting 40% of the total mappers will be running in opportunistic containers. Once the job is running, navigate to `http://<resource-manager>:<webapp-port>/cluster/nodes`. We can see a column `QueuedContainers` where as the value becomes larger than 0. That means node manager starts to queue opportunistic containers.

![Queued Opportunistic Containers](/assets/queued_containers.jpg "Queued Opportunistic Containers")

In this simple test, my single node cluster has no free resource to launch these opportunistic containers, therefore I start to see following messages in job execution log,

```
2017-11-29 11:47:59,579 INFO mapreduce.Job: Task Id : attempt_1511926523092_0003_m_000036_0, Status : FAILED
[2017-11-29 11:47:57.597]Container Killed to make room for Guaranteed Container.
[2017-11-29 11:47:57.616]Container killed on request. Exit code is 143
[2017-11-29 11:47:57.633]Container exited with a non-zero exit code 143.
```

this task I have launched `100 mappers`, and 40% of them `40` are opportunistic containers, so there were `40 task attempts` been killed to make room for guaranteed containers. Only after all the guaranteed containers successfully executed, these opportunistic containers got the resource to run.
