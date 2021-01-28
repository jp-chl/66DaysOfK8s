# Day 9 of #66DaysOfK8s

_Last update: 2021-01-19_

---

Today, I have started a series of lessons in order to review the Kubernetes Architecture.
In this 1st part, a focus is on the master node components.

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

* The main components of a **master** node

---

### The main components of a master node

The master runs several processes:

* **kube-apiserver**: All internal and external traffic is handled by this process. Also, this is the only component that connects to the **etcd** database. It acts as the frontend of the cluster's shared state.
* **kube-scheduler**: This component is responsible for assigning Pods to Nodes based on available resources. It uses its own algorithm, however, it can be customized with [quota restrictions](https://kubernetes.io/docs/concepts/policy/resource-quotas), [taints and tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/), [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/), etc. You can even use a [custom scheduler](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/).
* **etcd**: This database handles the state of the cluster, networking and other persistent information. It is a b+tree key-value store.

![Control plane components](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

<div align="center" ><i><a target="_blank"  href="https://kubernetes.io/docs/concepts/overview/components/">kubernetes.io image</a></i></div>


---

Additional components:

* The **kube-controller-manager**, a core control loop daemon that determines the state of the cluster by communicating with the **kube-apiserver**. It watches the shared state of the cluster, and matches the desired state with the current one, if necessary. There are several controllers such as _node controller_, _replication controller_, _endpoints controllers_ and, _service account & token controller_ (notices when nodes go down, maintains the correct number of pods, joins services and pods and, handles access for new namespaces; respectively).
* The **cloud-controller-manager** embeds cloud-specfic control logic, and links the cluster with cloud provider's API. _Node_, _Route_ and _Service_ controllers can have cloud dependencies (_deleted nodes in the cloud, setting up routes in the cloud infrastructure and, managing cloud provider load balancers; respectively_).

> _The cloud-controller-manager handles tasks once managed by the kube-controller-manager._

---

## References

* [Part 2: Worker node components](../day10)

* [Cluster Architecture (official site)](https://kubernetes.io/docs/concepts/overview/components/)

* [Kubernetes Architecture explained (15 minutes video)](https://www.youtube.com/watch?v=umXEmn3cMWY&ab_channel=TechWorldwithNana)

* [Kubernetes Architecture Explained in Brief](https://medium.com/swlh/kubernetes-architecture-explained-in-brief-6a07f59193e)

* [Simplified Kubernetes Architecture](https://medium.com/@mohan08p/simplified-kubernetes-architecture-3febe12480eb)

* [The Kubernetes Cluster Architecture Simplified](https://medium.com/dev-genius/the-kubernetes-cluster-architecture-simplified-3c4a5fb41449)

* [Under the Hood: An Introduction to Kubernetes Architecture](https://medium.com/@yashbindlish1/under-the-hood-an-introduction-to-kubernetes-architecture-bb9d8599f837)

* [Kubernetes in 10 Minutes: A Complete Guide](https://medium.com/faun/kubernetes-in-10-minutes-a-complete-guide-a9230124a02c)

* [Kubernetes Fundamentals For Absolute Beginners: Architecture & Components](https://medium.com/the-programmer/kubernetes-fundamentals-for-absolute-beginners-architecture-components-1f7cda8ea536)
