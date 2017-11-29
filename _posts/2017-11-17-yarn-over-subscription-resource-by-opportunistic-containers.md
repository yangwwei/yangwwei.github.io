---
title:  Over-Subscription Cluster Resource by Opportunistic Containers
tags:
  - YARN
---

Just like airline overbooks their flights, yarn can (and should) provide the capability
to overbook its resource for applications to increase the resource utilization.

<!--more-->

### Motivation

One problem with yarn is that we can hardly get the cluster to reach a relative
high resource utilization due to its scheduling model. Majorly because,

1. Yarn containers are fixed size, once allocated, it occupies the fixed amount
of resource during its entire lifecycle. However, resource usage of an application
is changing over the time, it might consume very rare resource when it is idle
so most of time the resource is wasted. This is even worse when there is long running services.
2. Most of user sets the resource request according to the maximum required
resource to ensure they get a predictable SLA for their applications. Again,
most of time such configuration wastes a lot of resources.

Opportunistic containers was added in [YARN-2882](https://issues.apache.org/jira/browse/YARN-2882), as a sub-task of [YARN-2877](https://issues.apache.org/jira/browse/YARN-2877). The over-subscription work is currently work-in-progress and tracked in [YARN-1011](https://issues.apache.org/jira/browse/YARN-1011). This post is exploring how this approach improves cluster resource utilization and how it works.

### Opportunistic Container

Opportunistic container is a new type of container that aims to over-subscribe system resource in order to increase resource utilization in a yarn cluster.

#### Guaranteed Container

Traditional container type, subscribe a fixed amount of resource from yarn cluster. Once a guaranteed container is allocated, the resource this container specified is booked from yarn capacity and yarn will ensure this amount of resource can be guaranteed at any point of time.

#### Opportunistic Container

The allocation decision for an opportunistic container is not depending on the capacity of the cluster, instead it is determined by the real time resource utilization in the cluster while the request is made. Which means, even the cluster capacity is all booked by guaranteed containers and there is no enough capacity to satisfy the desired requested resource of an
opportunistic container, it can be still allocated as long as there is free resource available on node managers. This is also called over-subscription.

This mechanism helps to improve the utilization of the resource. However, it is necessarily to know that yarn gives no guarantee of the resources allocated to such containers. So it might be killed or preempted when it needs to recycle some resource to avoid starving guaranteed containers.

![Chart 1. Over-subscription](/assets/yarn-over-subscription-1.jpg)

Chart 1 illustrates how the over-subscription works on a node manager. Node manager monitors the node resource usage in a certain frequency (default 3s), if it detects there is free resource available on this node, then these resource can be utilized by opportunistic containers. The total allocated the resource cannot exceed the system limit at any time.

#### Schedule Opportunistic Containers

Resource manager runs an application master service processor to handle the requests from application masters, it was working like following

![Char 2 - Current Scheduling Logic](/assets/yarn-over-subscription-2.jpg)

Procedure:

1. Application master submits resource requests to the application master service running in the resource manager
2. Application master service queries a specific scheduler to decide how to schedule containers
3. Resource manager returns the scheduling decision back to application master
4. Application master interpretes the allocation response and connects to specific node managers to launch containers

With adding the concept of opportunistic containers, the procedure now looks like

![Char 3 - Scheduling Guaranteed and Opportunistic Containers](/assets/yarn-over-subscription-3.jpg)

Rest of processes remain the same, except for

* There is a chain of application master service processors, the default one handles the scheduling of guaranteed containers and the opportunistic AMS processor handles the scheduling of opportunistic containers

#### Over-allocation and Preemption

Node manager over-allocation and preemption is controlled by thresholds, they are configurable percentage of nodes’ resource.

![Chart 4 - Over-allocation and Preemption](/assets/yarn-over-subscription-4.jpg)

Node manager collects the memory/CPU usage by all containers from cgroups, and reports that to resource manager via heartbeat. There are two configurable thresholds, one for over-allocation and the other for preemption. When node resource utilization is under the over-allocation-threshold, this node will be scheduled with some opportunistic containers that utilizes the resource beyond this threshold and under the preemption-threshold.

If the resource usage exceeds the preemption-threshold, then node manager starts to preempt containers from this node to keep the utilization under this threshold.
