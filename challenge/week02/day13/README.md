# Day 13 of #66DaysOfK8s

_Last update: 2021-01-23_

---

Today, I have worked in part 5 of a series of lessons in order to review the Kubernetes Architecture.
On this 5th day, a focus is on Pods.

#kubernetes #learning #K8s #66DaysChallenge

---

## TL;DR

_"A Pod is a group of one or more application containers (such as Docker) and includes shared storage (volumes), IP address and information about how to run them."_ (official Kubernetes documentation)

![Pods overview](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

---

## Versions used

* N/A (only theory)

---

## Setup

* N/A

---

## Tasks

Understand:

* Kubernetes Pod

---

### Pods

A Pod is a group of one or more containers, with shared storage and network resources

A Pod can contain [init containers](https://github.com/jp-chl/66DaysOfK8s/tree/master/challenge/week02/day12) that run during Pod startup, or also [ephemeral containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/) (mainly for debugging).

There is only one IP address per Pod, for almost every [network plugin](https://kubernetes.io/docs/concepts/cluster-administration/networking/). Multiple containers in a Pod share the same IP. To communicate with each other, there are some options like [IPC, the loopback interface or a shared filesystem](https://thenewstack.io/review-of-container-to-container-communications-in-kubernetes/).

---

Normally a Pod is created with workload resources like a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) or a Job. Also, with a StatefulSet resource if state tracking is needed.

A Pod can run a single container (most common use case), or multiple containers share resources (storage, networking) within the same Pod. In the latter, usually one container is an application and other can be a container that handles tasks like logging or traffic ([sidecar pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar)).

A replication is the process of scaling the application horizontally (i.e. run multiple Pods of the same app). Workload resources, like [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) and [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset), handle replication and rollout of multiple Pods.

---

All Pods are scheduled from the Control Plane except [Static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/).

---

## References

* [Pods (official site)](https://kubernetes.io/docs/concepts/workloads/pods/)

* [Sidecar pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar)

* [Review of Container-to-Container Communications in Kubernetes](https://thenewstack.io/review-of-container-to-container-communications-in-kubernetes/)
