---
title: Software Engineer, Real-time Data Infrastructure, Alibaba Group
duration: Sep, 2017 - Aug, 2018
skills:
  - HDFS
  - YARN
  - Docker
  - Flink
  - HBase
  - Java
  - Python
---

Our team is building [Blink](https://data-artisans.com/blog/blink-flink-alibaba-search) (a fork from apache [Flink](https://data-artisans.com/blog/blink-flink-alibaba-search)) to serve Alibaba's large scale, real-time streaming processing. We are dedicated to build world-best computation
engine for both real-time streaming and batch job processing. My current focus is evolving the capability to manage resource for both offline and online applications in [Apache Yarn](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html), improve the general compatibility for various of workloads and improve the resource utilization of big clusters. Primary work done includes

* Collaborate with [Wangda Tan](https://www.linkedin.com/in/wangdatan/), [Konstantinos Karanasos](https://www.linkedin.com/in/kkaranasos/) and [Arun Suresh](https://www.linkedin.com/in/arunsuresh/) on rich placement constraints support in YARN, see [YARN-6592](https://issues.apache.org/jira/browse/YARN-6592) and [YARN-7812](https://issues.apache.org/jira/browse/YARN-7812). And deployed the feature on production clusters, it helped our users to achieve multi-level affinity/anti-affinity placement requirements.
* Collaborate with [Sunil Govindan](https://www.linkedin.com/in/sunilgovindan/), [Naganarasimha Garla](https://www.linkedin.com/in/naganarasimha-garla-a620297/) on implementing YARN node attributes, see [YARN-3409](https://issues.apache.org/jira/browse/YARN-3409). This feature is currently used by our service jobs for node-binding hard constraint placements. E.g avoid port conflicts, volume/disk anti-affinity etc.
* Internally worked with Alibaba engineers to fully implemented Global Scheduling for Capacity Scheduler, based on [YARN-5139](https://issues.apache.org/jira/browse/YARN-5139). Features including per-policy sorting nodes service, pluggable policies, async scheduling (decoupled with HBs). We have developed a score-based policy to score nodes by multiple criteria, such as its resource usage, resource weights. We have also done some enhancements on sorting & reservation mechanism to reduce proposal conflicts. This is currently deployed in our production clusters, it helped to balance the loads.

We have summarized the work we've done and presented to the crowd during Dataworks Summit 2018, check it out [here](https://www.slideshare.net/Hadoop_Summit/apache-hadoop-yarn-3x-in-alibaba).
