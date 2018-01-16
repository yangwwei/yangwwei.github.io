---
title: SchedulingRequest-handling-in-YARN
tags:
  - YARN
  - PlacementConstraint
---

Some reading notes about how scheduling request is handled in YARN on the master branch.
For Chinese reader ONLY.
<!--more-->

注意：当前的工作还在进行中，该文档可能随时被更新。

### 处理SchedulingRequest的两种途径

* PlacementProcessor

参见[YARN-7612](https://issues.apache.org/jira/browse/YARN-7612)
该途径是通过在allocate的链路上前置若干个placement的processor来处理单纯的与node-attribute/tag相关的placement的约束。这些约束与application的priority、queue的capacity等无直接联系。

* 通过 AppPlacementAllocator

参见[YARN-6599](https://issues.apache.org/jira/browse/YARN-6599) 该途径将对scheduling request的处理嵌入到了已有的调度逻辑中。由于在scheduler的context里面，这种方法可以支持node-partition作为constraint的target，并且能够尊重application的priority、queue的ordering等一些scheduler内部的约束。
