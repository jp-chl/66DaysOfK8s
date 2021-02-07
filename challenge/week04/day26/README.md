# Day 26 of #66DaysOfK8s

_Last update: 2021-02-05_

---

Today, I have worked with Replica sets.

#kubernetes #learning #K8s #66DaysChallenge


---

## TL;DR

This is a practical exercise as a first approach to a ReplicaSet controller in K8s. It manages Pods.

[Demo](#demo)

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Setup

* All tests run on minikube.

---

## Tasks

* Test examples of Replica sets.


---

### Test examples of Replica sets

Let's create a simple ReplicaSet.

```yaml
# rs.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-rs # Replica name
spec:
  replicas: 2 # Two Pods
  selector:
    matchLabels: 
      system: MyReplica # Selector label key/pair (i.e. "system"/"MyReplica")
  template:
    metadata:
      labels:
        system: MyReplica
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```bash
$ kubectl apply -f yaml/rs.yaml
replicaset.apps/my-rs created
```

```bash
$ kubectl get pods,rs
```

```bash
$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
my-rs-9hpnp   1/1     Running   0          97s
my-rs-gtz64   1/1     Running   0          97s
```

---

Let's choose one of the two Pods.

```bash
$ kubectl get pods -l "system=MyReplica" --output=jsonpath='{.items[0].metadata.name}'
my-rs-9hpnp
```

Edit the Pod and change the selector label to "```MySinglePod```" in order to release it from the ReplicaSet control.

```bash
$ kubectl edit pod $(kubectl get pods -l "system=MyReplica" --output=jsonpath='{.items[0].metadata.name}')
```

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-02-05T22:49:25Z"
  generateName: my-rs-
  labels:
    system: MyReplica # change to "MySinglePod"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:conditions:
          k:{"type":"ContainersReady"}:
            {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
```

After saving the Pod spec, a third Pod will be created.

```bash
$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
my-rs-9hpnp   1/1     Running   0          3m16s
my-rs-gtz64   1/1     Running   0          3m16s
my-rs-j4tgn   1/1     Running   0          18s
```

Two Pods are still under ReplicaSet control.

```bash
$ kubectl get pods -l "system=MyReplica"
NAME          READY   STATUS    RESTARTS   AGE
my-rs-gtz64   1/1     Running   0          3m20s
my-rs-j4tgn   1/1     Running   0          22s
```

And a new one has been created.

```bash
$ kubectl get pods -l "system=MySinglePod"
NAME          READY   STATUS    RESTARTS   AGE
my-rs-9hpnp   1/1     Running   0          3m27s
```

---

If you delete the ReplicaSet, the two Pods will be evicted however the newly created Pod won't.

```bash
$ kubectl delete rs my-rs
replicaset.apps "my-rs" deleted
```

```bash
$ kubectl get pods -l "system=MyReplica"
No resources found in default namespace.
```

```bash
$ kubectl get pods -l "system=MySinglePod"
NAME          READY   STATUS    RESTARTS   AGE
my-rs-9hpnp   1/1     Running   0          4m8s
```

---

Delete the single Pod.

```bash
$ kubectl delete pods -l "system=MySinglePod"
pod "my-rs-9hpnp" deleted
```

```bash
$ kubectl get pods,rs
No resources found in default namespace.
```

---

## References

* [ReplicaSet (official site)](https://kubernetes.io/es/docs/concepts/workloads/controllers/replicaset/)

---

# Demo

[![asciicast](https://asciinema.org/a/1yxMKzopY3EwkNgRg4wt7pWYE.svg)](https://asciinema.org/a/1yxMKzopY3EwkNgRg4wt7pWYE)
