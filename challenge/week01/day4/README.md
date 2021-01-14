# Day 4 of #66DaysOfK8s

_Last update: 2021-01-14_

---

Today, I've learned to use Pod's priorities. The priority indicates the importance of a Pod relative to other Pods.

> _Based on: [https://medium.com/faun/kubernetes-cka-hands-on-challenge-6-pod-priority-1fe95f613ac5](https://medium.com/faun/kubernetes-cka-hands-on-challenge-6-pod-priority-1fe95f613ac5)_

#kubernetes #learning #K8s #66DaysChallenge

---

## TL;DR

[Demo](#demo)

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.18.9

---

## Setup

* All tests runs on minikube.
* Pods are deployed on different namespaces
* 1 node (worker) with 2GB Ram

---

## Tasks

1. Compare priorites of pods

2. Attempt to allocate resources for a Pod according to its priority

---

## Create lab

```bash
minikube config set memory 2048
```

Start minikube

```bash
minikube start --kubernetes-version=v1.18.9
```

Create a new namespace called "management"

```bash
$ kubectl create ns management
namespace/management created
```

---

## Compare priorites of pods

By default, K8s has ```PriorityClass``` objects with maximum priority (0 is the lowest).
> _More info at the official documentation [in this link](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/)._

```bash
$ kubectl get priorityclasses
NAME                            VALUE        GLOBAL-DEFAULT   AGE
system-cluster-critical         2000000000   false            172m
system-node-critical            2000001000   false            172m
```

In this case, system-cluster-critical and system-node-critical have a priority of 2000000000.

Let's create a new one (```priorityclass-important-pods.yaml```) in order to assign it to a pod (later on).

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: priority-class-important-pods
value: 1000000 # this is an example, any number greater than zero
preemptionPolicy: Never
globalDefault: false
description: 'priority class important pods'
```

```bash
$ kubectl create -f priorityclass-important-pods.yaml
priorityclass.scheduling.k8s.io/priority-class-important-pods created
```

```bash
$ kubectl get priorityclasses
NAME                            VALUE        GLOBAL-DEFAULT   AGE
priority-class-important-pods   1000000      false            26s
system-cluster-critical         2000000000   false            179m
system-node-critical            2000001000   false            179m
```

---

Now, let's create a Pod (```less-important-pod.yaml```), in management namespace, with the lowest priority:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: less-important-pod
    app: less-important-pod
  name: less-important-pod
  namespace: management
spec:
  containers:
  - image: nginx:1.17.6-alpine
    name: less-important-pod
  restartPolicy: Never
  #priorityClassName: # does not have any, i.e., zero
status: {}
```

```bash
$ kubectl -n management apply -f less-important-pod.yaml
pod/less-important-pod created
```

We expect its priority to be zero:

```bash
$ kubectl -n management get pods -l "app=less-important-pod" -o json -o jsonpath='{.items[0].spec.priority}'
0
```

Now, let's create a new pod (```important-pod.yaml```) with a higher priority (based on PriorityClass already defined):

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: important-pod
    app: important-pod
  name: important-pod
  namespace: management
spec:
  containers:
  - image: nginx:1.17.6-alpine
    name: important-pod
  restartPolicy: Never
  priorityClassName: priority-class-important-pods # PriorityClass
status: {}
```

```bash
$ kubectl -n management apply -f important-pod.yaml
pod/important-pod created
```

In namespace management only one pod will have PriorityClass.

```bash
$ kubectl -n management get pod -o jsonpath='{range .items[*]}{.metadata.name}: {.spec.priorityClassName}{"\n"}'
important-pod: priority-class-important-pods
less-important-pod:
```

---

Create a new Pod (```not-so-imporant-pod.yaml```) in default namespace and add 1.5 Gi of memory requests:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: not-so-important-pod
    app: not-so-important-pod
  name: not-so-important-pod
  #namespace: default
spec:
  containers:
  - image: nginx:1.17.6-alpine
    name: not-so-important-pod
    ## -----------------------
    resources: 
      requests:
        memory: 1.5Gi
    ## -----------------------
  restartPolicy: Never
  #priorityClassName: # does not have any, i.e., zero
status: {}
```

```bash
$ kubectl -n default apply -f not-so-imporant-pod.yaml
pod/not-so-important-pod created
```

---

## Attempt to allocate resources for a Pod according to its priority

Based on ```not-so-imporant-pod.yaml```, create a similar yaml (```very-much-so-important.yaml```) with different labels but with the same memory requests.

```yaml
# Identical to not-so-important-pod.yaml but its labels
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: very-much-so-important
    app: very-much-so-important
  name: very-much-so-important
  #namespace: default
spec:
  containers:
  - image: nginx:1.17.6-alpine
    name: very-much-so-important
    ## -----------------------
    resources: 
      requests:
        memory: 1.5Gi
    ## -----------------------
  restartPolicy: Never
  #priorityClassName: # does not have any, i.e., zero
status: {}
```

If we try to apply this (very-much-so-important) Pod it will keep in "Pending" status until more resources are available.

```bash
$ kubectl -n default apply -f  very-much-so-important.yaml
pod/very-much-so-important created
```

```bash
$ kubectl -n default get pods
NAME                     READY   STATUS    RESTARTS   AGE
not-so-important-pod     1/1     Running   0          10m
very-much-so-important   0/1     Pending   0          50s
```

If we check pod's last event we'll notice an "insufficient memory" message; remember, we have built a 2 Gi node.

```bash
$ kubectl get event -n default --field-selector involvedObject.name=very-much-so-important
LAST SEEN   TYPE      REASON             OBJECT                       MESSAGE
64s         Warning   FailedScheduling   pod/very-much-so-important   0/1 nodes are available: 1 Insufficient memory.
```

> _We could have also seen its events by running ```kubectl describe``` command (i.e. kubectl -n default describe pod very-much-so-important)_

---

In order to create the last Pod (very-much-so-important), we could assing the, already created, PriorityClass to it.

Let's modify Pod definition:

```yaml

# Identical to not-so-important-pod.yaml but its labels
apiVersion: v1
kind: Pod
metadata:

# ...

name: very-much-so-important
spec:

# ...

  #priorityClassName: # does not have any, i.e., zero
  priorityClassName: priority-class-important-pods
status: {}
```

Changes won't be effective (i.e. ```kubectl apply```...) unless we delete the pod and create it again.

```bash
$ kubectl -n default apply -f very-much-so-important.yaml
The Pod "very-much-so-important" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
...
```

```bash
$ kubectl -n default delete -f very-much-so-important.yaml
pod "very-much-so-important" deleted
```

```bash
$ kubectl -n default apply -f very-much-so-important.yaml
pod/very-much-so-important created
```

The ```very-much-so-important``` Pod now is running, and the ```not-so-important-pod``` has been terminated.

```bash
$ kubectl -n default get pods
NAME                     READY   STATUS    RESTARTS   AGE
very-much-so-important   1/1     Running   0          75s
```

```bash
$ kubectl get event -n default --field-selector involvedObject.name=not-so-important-pod
LAST SEEN   TYPE     REASON      OBJECT                     MESSAGE
28m         Normal   Scheduled   pod/not-so-important-pod   Successfully assigned default/not-so-important-pod to minikube
28m         Normal   Pulled      pod/not-so-important-pod   Container image "nginx:1.17.6-alpine" already present on machine
28m         Normal   Created     pod/not-so-important-pod   Created container not-so-important-pod
28m         Normal   Started     pod/not-so-important-pod   Started container not-so-important-pod
2m4s        Normal   Killing     pod/not-so-important-pod   Stopping container not-so-important-pod
2m4s        Normal   Preempted   pod/not-so-important-pod   Preempted by default/very-much-so-important on node minikube
```

---

# Demo

[![asciicast](https://asciinema.org/a/cjBkOAPiB9nmGrFECxUPzTYGV.svg)](https://asciinema.org/a/cjBkOAPiB9nmGrFECxUPzTYGV)
