# Day 10 of #66DaysOfK8s

_Last update: 2021-01-20_

---

Today, I have worked in part 2 of a series of lessons in order to review the Kubernetes Architecture.
On this 2nd day, a focus is on the worker node components and especially on the Kubelet.

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

* The main components of a **worker** node
* Kubelet

---

### The main components of a worker node

All nodes run the **kubelet** and **kube-proxy**, as well as the **container runtime**, such as Docker or cri-o.

* **Kubelet**: This component watches the containers that need to be running by communicating with the underlying Docker Engine (also installed on all the nodes), and taking a set of _PodSpecs_ into consideration.
* **Kube-proxy**: Network proxy than handles network rules. It manages the network connectivity among the containers (inside or outside of the cluster). It also monitors Services and Endpoints using a random port to proxy traffic. Besides, it uses the operating system packet filtering layer or, it forwards the traffic itself if it isn't available.
* **Container runtime**: This artifact is responsible for running containers. It supports several container runtimes such as Docker, [containerd](https://landscape.cncf.io/?selected=containerd), [cri-o](https://landscape.cncf.io/?selected=cri-o) (and any [Kubernetes Container Runtime Interface](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) implementation).
> _The "container runtime" is also called "container engine"_

**Addons**:

* These additional components use Kubernetes resources ([DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/), [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), etc) to implement cluster features. They run in the ```kube-system``` namespace.
* All Kubernetes cluster should have a [cluster DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), which is a server that provides DNS records for K8s services.
* An extended list of available addons can be found in this [link](https://kubernetes.io/docs/concepts/cluster-administration/addons/).

[Supervisord](http://supervisord.org/) is a process monitor that can be used (in non-[systemd](https://en.wikipedia.org/wiki/Systemd) cluster) to watch for the _kubelet_ and _docker_ processes. It will restart them if they fail, and log its events. Nevertheless, it isn't part of a typical installation.

K8s doesn't have cluster-wide logging (yet). Normally, [Fluentd](https://landscape.cncf.io/?selected=fluentd) is used (and lastly, [Loki](https://grafana.com/oss/loki/) is a good alternative). Those frameworks provide a unified logging layer for the cluster.

Cluster-wide metrics is another area with limited functionality. The [metrics-server SIG](https://github.com/kubernetes-sigs/metrics-server) provides basic node and pod information, however, [Prometheus](https://landscape.cncf.io/?selected=prometheus) is widely used.

---

### Kubelet

This is the primary "node agent", running on every worker, that watches for changes and configuration on them. It registers the node with the apiserver.
It accepts the API calls for Pod specification (a [PodSpec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#podspec-v1-core) is a ```yaml``` or ```json``` object that describes a pod), and ensures the specification has been met (containers are running and healthy).

Mainly:
* Mounts volumes to Pod (storage).
* Handles [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) or [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/).
* Passes requests to local container runtime.
* Reports status of Pods and node to the kube-apiserver for [eventual persistence](https://downey.io/blog/desired-state-vs-actual-state-in-kubernetes/#:~:text=Since%20Kubernetes%20is%20optimized%20for,eventually%2C%20the%20system%20will%20converge.).


There are several components that handle optimizations (such as CPU isolation, memory and device locality). The [Topology Manager](https://kubernetes.io/docs/tasks/administer-cluster/topology-manager/#:~:text=The%20Topology%20Manager%20is%20a,send%20and%20receive%20topology%20information) (a Kubelet component) co-ordinate this set of optimization components using _hints providers_, an interface for components to send and receive topology information. Kubelet components can make topology aligned resource allocation choices by using these hints as a source of truth.
As an alpha feature, it isn't enabled by default.

---

## References

* [Part 3: Kubernetes Controllers](../day11)

* [Cluster Architecture (official site)](https://kubernetes.io/docs/concepts/overview/components/)

* [Kubernetes Architecture explained (15 minutes video)](https://www.youtube.com/watch?v=umXEmn3cMWY&ab_channel=TechWorldwithNana)

* [Kubernetes Architecture Explained in Brief](https://medium.com/swlh/kubernetes-architecture-explained-in-brief-6a07f59193e)

* [Simplified Kubernetes Architecture](https://medium.com/@mohan08p/simplified-kubernetes-architecture-3febe12480eb)

* [The Kubernetes Cluster Architecture Simplified](https://medium.com/dev-genius/the-kubernetes-cluster-architecture-simplified-3c4a5fb41449)

* [Under the Hood: An Introduction to Kubernetes Architecture](https://medium.com/@yashbindlish1/under-the-hood-an-introduction-to-kubernetes-architecture-bb9d8599f837)

* [Kubernetes in 10 Minutes: A Complete Guide](https://medium.com/faun/kubernetes-in-10-minutes-a-complete-guide-a9230124a02c)

* [Kubernetes Fundamentals For Absolute Beginners: Architecture & Components](https://medium.com/the-programmer/kubernetes-fundamentals-for-absolute-beginners-architecture-components-1f7cda8ea536)
