---
title: SchedulingRequest Handling in YARN
tags:
  - YARN
  - PlacementConstraint
---

Some reading notes about how scheduling request is handled in YARN on the master branch.
For Chinese reader ONLY.
<!--more-->

注意：当前的工作还在进行中，该文档可能随时被更新。

`SchedulingRequest` 是为了增加请求中携带约束条件而新增的请求类型，与`ResourceRequest`为平行的关系。应用如果想在请求中指定相关的约束条件，则需要发送SchedulingRequest。在Yarn中，处理SchedulingRequest有两种途径。

### 处理SchedulingRequest的两种途径

* PlacementProcessor

参见[YARN-7612](https://issues.apache.org/jira/browse/YARN-7612)
该途径是通过在allocate的链路上前置若干个placement的processor来处理单纯的与node-attribute/tag相关的placement的约束。这些约束与application的priority、queue的capacity等无直接联系。该途径的好处是处理逻辑相对简单，与调度器代码偶合较少，维护成本较低。但是由于调度流程中影响约束条件的因素较多而且变化较大，完全在前置的链路处理似乎并不可能。

* 通过 AppPlacementAllocator

参见[YARN-6599](https://issues.apache.org/jira/browse/YARN-6599) 该途径将对scheduling request的处理嵌入到了已有的调度逻辑中。由于在scheduler的context里面，这种方法可以支持node-partition作为constraint的target，并且能够尊重application的priority、queue的ordering等一些scheduler内部的约束。该途径能够更好的与调度流程兼容，也更能确保约束条件的正确性。
