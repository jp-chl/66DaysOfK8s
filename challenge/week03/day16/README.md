# Day 16 of #66DaysOfK8s

_Last update: 2021-01-26_

---

Today, I have worked in the last part of a series of lessons in order to review the Kubernetes Architecture.
On this 8th day, a focus is on Cluster Networking.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* N/A (only theory)

---

## Setup

* N/A

---

## Tasks

Understand:

* Cluster Networking

---

### Networking setup

* A Pod can be seen as a virtual machine or physical host (w.r.t. port allocation, service discovery, etc.). The network has to assign an IP address to any Pod, and route traffic between all Pods (regardless of which node it is located).

* A container orchestration system has these networking issues to deal with:
  * Highly coupled container-to-container communication within the same Pod (solved by [Pod networking](https://github.com/jp-chl/66DaysOfK8s/tree/master/challenge/week03/day15)).
  * Pod-to-Pod communication.
  * Service-to-Pod communication and External-to-Service communication (solved by the [services](https://kubernetes.io/docs/concepts/services-networking/service/) concept).

* The container networking is standardized on the Container Network Interface ([CNI](https://github.com/containernetworking/cni)) specification, which configures it and removes allocated resources when the container is deleted. It provides a common interface between network solutions and container runtimes.

* CNI helps to provide a single IP per Pod, however, to manage Pod-to-Pod communication Kubernetes delegates this to a network model/implementation which has to follow these requirements:
  * Pods on a node can communicate with all Pods on any node without NAT.
  * Agents on any node (e.g. [daemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/), [kubelet](https://github.com/jp-chl/66DaysOfK8s/tree/master/challenge/week02/day10)) can communicate with all Pods on it.

* There are several alternatives that implement the networking model (check a [list in the official site](https://kubernetes.io/docs/concepts/cluster-administration/networking/), for example:  [Weave](https://www.weave.works/products/weave-net/), [Flannel](https://github.com/coreos/flannel#flannel), [Calico](https://docs.projectcalico.org/), etc.

---

## References

* [Cluster networking (official site)](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

* [Kubernetes Networking Guide for Beginners](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-networking-guide-beginners.html)