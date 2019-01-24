---
title: Setup Local Kubernetes on Mac
tags:
  - Kubernetes
---
A tutorial about how to setup a local k8s cluster on MacOS for development.
<!--more-->

### Setting local k8s environment with docker-desktop

1. Install `Docker` on Mac with this [instruction](https://docs.docker.com/docker-for-mac/install/), latest version of docker embedded native support for Kubernetes.
2. Start docker, once started, you'll see `Docker is running` and `Kubernetes is running`.
3. Ensure Kubernetes is started with `docker-for-desktop` option, select from `docker -> Kubernetes-> docker-for-desktop`.
4. Set kubectl's context to `docker-for-desktop`
```
kubectl config use-context docker-for-desktop
```
otherwise, kubectl will not be able to connect to the local cluster.
5. Install k8s dashboard,
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml -v=8
```
6. Check pod status
```
kubectl get pod --namespace=kube-system | grep dashboard
kubernetes-dashboard-669f9bbd46-p4f7c        1/1       Running   1          1h
```
7. Access dashboard
```
kubectl proxy
# URL
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login
```

8. Login with ACL
```
kubectl -n kube-system get secret
kubectl -n kube-system describe secrets kubernetes-dashboard-token-tf6n8
```

### Backup

1. Delete dashboard service
```
kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```
