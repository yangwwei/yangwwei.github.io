---
title: Kubernetes Scheduler
tags:
  - Kubernetes
  - Docker
  - Containerize
  - CNCF
---
This post introduces Kubernetes scheduler from a newbie's narrative.
<!--more-->

## Namespace

K8s uses namespace to divide a physical cluster to multiple virtual clusters. Together with `Resource Quota`, k8s can divide cluster resources between multiple users.

## Resource Quota

`ResourceQuota` object provides constraints that limit aggregated resource consumption per `namespace`. It supports

1. Set `Compute Resource Quota`, e.g memory/cpu/gpu etc
2. Set `Storage Resource Quota`, e.g volume claims, persistent volume claims etc
3. Set `Object Count Quota`, this allows user to set limits for various of object counts, such as number of services, deployment/relicaset apps etc.
4. Quota can have an associated `Scope`, available scope includs `Terminating`, `NotTerminating`, `BestEffort`, `NotBestEffort`.
5. `ResourceQuota` can be created per `PriorityClass`

Limitations

1. Resource quotas are are independent of the cluster capacity. When nodes added/removed from k8s cluster, quotas won't be updated dynamically.
2. Doesn't allow tenant to grow resource usage as needed, but have a generous limit to prevent accidental resource exhaustion.

Here is an example of Resource quota

```
// mem-cpu-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

Create the ResourceQuota

```
kubectl create -f mem-cpu-quota.yaml --namespace=quota-mem-cpu-example
```

## Preemption

When a pod cannot be scheduled, k8s scheduler can preempt (evict) lower priority pods to make room for the pending pods.

[Detail](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#preemption): When Pods are created, they go to a queue and wait to be scheduled. The scheduler picks a Pod from the queue and tries to schedule it on a Node. If no Node is found that satisfies all the specified requirements of the Pod, preemption logic is triggered for the pending Pod. Let’s call the pending Pod P. Preemption logic tries to find a Node where removal of one or more Pods with lower priority than P would enable P to be scheduled on that Node. If such a Node is found, one or more lower priority Pods get deleted from the Node. After the Pods are gone, P can be scheduled on the Node.

Limitations:
1. [Graceful termination of preemption victims](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#graceful-termination-of-preemption-victims): a preempted pod needs sometime to gracefully termination, that creates a time gap that other pods might be scheduled on this node causing an unsuccessful preemption.
2. [PodDisruptionBudget is supported, but not guaranteed!](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#poddisruptionbudget-is-supported-but-not-guaranteed): [PDB](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) can be set to avoid pod's disruption across replicas, however, this is a best-effort budget.
3. [Inter-Pod affinity on lower-priority Pods](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#inter-pod-affinity-on-lower-priority-pods): if a pod has affinity constraint to a lower priority pod on this node, this node might not be a candidate for preemption.
4. [Cross node preemption](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#cross-node-preemption): scheduler doesn't consider cross-node preemption yet.
