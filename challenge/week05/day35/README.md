# Day 35 of #66DaysOfK8s

_Last update: 2021-02-14_

---
Today, I have worked with taints and tolerations.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client
* kubectl client and server v1.19.0

---

## Setup

* K8s cluster already created in GCP from scratch. Check the instructions in this [link](../../week01/day5/README.md).
* Set an alias for kubectl (```alias k=kubectl```).
* All tests are done in default namespace.

---

## Tasks

* Taint a node.
* Create a pod with no toleration to taints.
* Create a pod that tolerates a taint.
* Remove the taint from a node in order to run the first pod.

---

### Taint a node

In this case, initially, both master and worker nodes have no taints.

```bash
$ student@master: k get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   29d   v1.19.0
worker   Ready    <none>   28d   v1.19.0
```

```bash
$ student@master: k describe node master |grep -i taint -A2
Taints:             <none>
Unschedulable:      false
```

```bash
$ student@master: k describe node worker |grep -i taint -A2
Taints:             <none>
Unschedulable:      false
```

Let's taint both nodes, with ```key/value``` ```node/worker``` and ```node/master```, for ```worker``` and ```master``` node, respectively. Additionally, these taints will be treated as ```NoSchedule```, i.e., unless a Pod has toleration for these tains, it won't be scheduled.

```bash
$ student@master: k taint nodes worker node=worker:NoSchedule
node/worker tainted
```

```bash
$ student@master: k describe node worker |grep -i taint -A1
Taints:             node=worker:NoSchedule
Unschedulable:      false
```

```bash
$ student@master: k taint nodes master node=master:NoSchedule
node/master tainted
```

```bash
$ student@master: k describe node master |grep -i taint -A1
Taints:             node=master:NoSchedule
Unschedulable:      false
```

---

### Create a pod with no toleration to taints

Let's create a simple Pod, called simple/pod. By default, a Pod has no tolerations. Therefore, this Pod won't be scheduled on any node.

```bash
$ student@master: k run simple-pod --image=nginx
pod/simple-pod created
```

```bash
$ student@master: k describe pod simple-pod| grep -i event -A6
Events:
  Type     Reason            Age   From  Message
  ----     ------            ----  ----  -------
  Warning  FailedScheduling  44s         0/2 nodes are available: 1 node(s) had taint {node: master}, that the pod didn't tolerate, 1 node(s) had taint {node: worker}, that the pod didn't tolerate.
  Warning  FailedScheduling  44s         0/2 nodes are available: 1 node(s) had taint {node: master}, that the pod didn't tolerate, 1 node(s) had taint {node: worker}, that the pod didn't tolerate.
```

The Pod's state will keep on Pending unless a toleration is added or a taint is removed.

```bash
$ student@master: k get pods
NAME         READY   STATUS    RESTARTS   AGE
simple-pod   0/1     Pending   0          2m2s
```

---

### Create a pod that tolerates a taint

Let's create another Pod, but specify a toleration in yaml format in order to be ```scheduled``` on the ```worker node```.

```bash
$ student@master: k run another-pod --image=nginx --dry-run=client -o yaml > yaml/anotherPod.yaml
```

```yaml
# anotherPod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null # Can be erased
  labels:
    run: another-pod
  name: another-pod
spec:
  containers:
  - image: nginx
    name: another-pod
    resources: {} # Can be erased
  dnsPolicy: ClusterFirst # Can be erased
  restartPolicy: Always # Can be erased
status: {} # Can be erased
```

Let's add the toleration section on this yaml.

```yaml
# anotherPod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: another-pod
  name: another-pod
spec:
  # ---------------
  tolerations:
  - key: node
    operator: Equal
    value: worker
    effect: NoSchedule
  # ---------------
  containers:
  - image: nginx
    name: another-pod
```

Apply it and check on which node the Pod runs.

```bash
$ student@master: k apply -f yaml/anotherPod.yaml
pod/another-pod created
```

```bash
$ student@master: k get Pods -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
another-pod   1/1     Running   0          10s   192.168.171.123   worker   <none>           <none>
simple-pod    0/1     Pending   0          10m   <none>            <none>   <none>           <none>
```

---

### Remove the taint from a node in order to run the first pod

Let's remove the taint on master node. To do it, add a ```-``` sign after the ```key```.

```bash
$ student@master: k taint nodes master node-
node/master untainted
```

```bash
$ student@master: k describe node master |grep -i taint -A1
Taints:             <none>
Unschedulable:      false
```

Check Pods status again. Now the first Pod will run on the ```master``` node.

```bash
$ student@master: k get Pods -o wide
NAME          READY   STATUS    RESTARTS   AGE     IP                NODE     NOMINATED NODE   READINESS GATES
another-pod   1/1     Running   0          3m33s   192.168.171.123   worker   <none>           <none>
simple-pod    1/1     Running   0          14m     192.168.219.127   master   <none>           <none>
```

---

### Cleanup

```bash
k delete pod simple-pod
k delete -f yaml/anotherPod.yaml

pod "simple-pod" deleted
pod "another-pod" deleted
```

---

## References

* [Taints and Tolerations (official site)](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

