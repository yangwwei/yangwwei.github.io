---
title: YARN-SLS-Guide
tags:
  - YARN
  - Performance
---

This post introduces how to run SLS tool to measure YARN performance.
<!--more-->

## Build Hadoop dist
From Hadoop source, build a dist package with following command:

```
mvn package -Pdist -DskipTests -Dtar -Dmaven.javadoc.skip=true -DskipShade
```

then extract Hadoop dist package to a folder, assume it is under `$HADOOP_DIST_ROOT`.

## Configure YARN

(1) Setup SLS conf

Sample configuration can be found in `share/hadoop/tools/sls/sample-conf`, copy them to `$HADOOP_DIST_ROOT/etc/hadoop`.

(2) Setup capacity scheduler

Configure `yarn-site.xml` and `capacity-scheduler.xml` based on the sample.

(3) Configure Logs

Configure `log4j.properties`, following configuration will print all logs in console

```
# Dump log to console
log4j.rootLogger=INFO,test
log4j.appender.test=org.apache.log4j.ConsoleAppender
log4j.appender.test.Target=System.out
log4j.appender.test.layout=org.apache.log4j.PatternLayout
log4j.appender.test.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n
log4j.logger=NONE, test

# To open DEBUG logging for a particular class
log4j.category.org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler=DEBUG
```

## Configure SLS

Configure job generator,

```
{
  "description": "tiny jobs workload",
  "num_nodes": 200,
  "nodes_per_rack": 4,
  "num_jobs": 100,
  "rand_seed": 2,
  "workloads": [
    {
      "workload_name": "tiny-test",
      "workload_weight": 0.5,
      "description": "Sort jobs",
      "queue_name": "sls_queue_1",
      "job_classes": [
        {
          "class_name": "class_1",
          "user_name": "foobar",
          "map_execution_type": "OPPORTUNISTIC",
          "class_weight": 1.0,
          "mtasks_avg": 5,
          "mtasks_stddev": 1,
          "rtasks_avg": 5,
          "rtasks_stddev": 1,
          "dur_avg": 60,
          "dur_stddev": 5,
          "mtime_avg": 10,
          "mtime_stddev": 2,
          "rtime_avg": 20,
          "rtime_stddev": 4,
          "map_max_memory_avg": 1024,
          "map_max_memory_stddev": 0.001,
          "map_execution_type": "GUARANTEED",
          "reduce_max_memory_avg": 2048,
          "reduce_max_memory_stddev": 0.001,
          "reduce_execution_type": "GUARANTEED",
          "map_max_vcores_avg": 1,
          "map_max_vcores_stddev": 0.001,
          "reduce_max_vcores_avg": 2,
          "reduce_max_vcores_stddev": 0.001,
          "chance_of_reservation": 0.5,
          "deadline_factor_avg": 10.0,
          "deadline_factor_stddev": 0.001
        }
        ],
      "time_distribution": [
        {
          "time": 1,
          "weight": 100
        },
        {
          "time": 60,
          "weight": 0
        }
      ]
    }
  ]
}
```

## Run SLS

```
cd $HADOOP_DIST_ROOT
./share/hadoop/tools/sls/bin/slsrun.sh --tracetype=SYNTH --tracelocation=../sync.json --output-dir=../sls-out
```

View Metrics

http://localhost:10001/simulate

View RM UI

http://localhost:18088/cluster/apps
