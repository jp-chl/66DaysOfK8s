# Day 11 of #66DaysOfK8s

_Last update: 2021-01-21_

---

Today, I have worked in part 3 of a series of lessons in order to review the Kubernetes Architecture.
On this 3rd day, a focus is on the Kubernetes Controllers.

#kubernetes #learning #K8s #66DaysChallenge

---

## TL;DR

A controller is a control loop that watches the shared state of the cluster through the API server and makes changes attempting to move the current state towards the desired state.
A simplified view of a controller is an agent, or Informer, and a downstream store. Using a _DeltaFIFO_ queue, the source and downstream are compared (_delta_ is any change in a resource).
The _Informer_ uses the apiserver as a source, and the data is cached to minimize transactions. A similar agent is the _SharedInformer_ which is used by objects interested in the same resource with a shared cache help.
A Workqueue uses a key to hand out tasks to various workers.
The endpoints, namespace, and serviceaccounts controllers each manage the resources for Pods.
Besides, you can build your own custom controller.

---

## Versions used

* N/A (only theory)

---

## Setup

* N/A

---

## Tasks

Understand:

* Kubernetes Controllers

---

### Kubernetes Controllers

As we've seen in the first part, A **controller** is a core control loop daemon that determines the state of the cluster by communicating with the **kube-apiserver**. It watches the shared state of the cluster, and matches the desired state with the current one, if necessary. There are several controllers such as _node controller_, _replication controller_, _endpoints controllers_ and, _service account & token controller_.

All controllers are packaged in a single daemon named **kube-controller-manager**. Its simple form of implementation is a loop:

```go
for {
  desired := getDesiredState()
  current := getCurrentState()
  makeChanges(desired, current)
}
```

There are two main components of a controller. An **Informer** (and a similar agent called the _SharedInformer_) and a **Workqueue**. The Informer watches for changes of the Kubernetes current state and sends events to the Workqueue. Finally, worker processes pop up the events from the queue.

---

### Informer

The controller sends a request to the apiserver in order to retrieve and object's state. Instead of calling many times a cache can be used. The informer component consumes events (by using a _Listwatcher_ interface) and gets notifications about creation, modification and deletion (with the help of a _Resource event handler_ component).

A _SharedInformer_ provides hooks for controllers that need notifications of the same resources making efficient cache usage (a single shared cache) and reducing memory overhead and apiserver calls.

---

### Workqueue

The controller must handle events in its own queue. Kubernetes library provides an efficient component called _workqueue_. When a resource changes, the Event Handler puts a key in the Workqueue (normally as _namespace/name_).
If a controller fails to process an event, it can push it back to the key to process it later. Otherwise, the event can be removed. 

---

## References

* [Part 4: Init containers](../day12)

* [A deep dive into Kubernetes controllers](https://engineering.bitnami.com/articles/a-deep-dive-into-kubernetes-controllers.html)

* [Understanding Kubernetes controllers part I – queues and the core controller loop](https://leftasexercise.com/2019/07/08/understanding-kubernetes-controllers-part-i-queues-and-the-core-controller-loop/)
