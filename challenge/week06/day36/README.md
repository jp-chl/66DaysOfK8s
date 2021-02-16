# Day 36 of #66DaysOfK8s

_Last update: 2021-02-15_

---
Today, I have worked with node affinity.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client
* kubectl client and server v1.19.0

---

## Setup

* K8s cluster already created in GCP from scratch (check the instructions in this [link](../../week01/day5/README.md)), or any cluster v 1.19 with at least two worker nodes.
* Set an alias for kubectl (```alias k=kubectl```).
* All Pods run a nginx image.

---

## Tasks

* Label a node.
* Assign a pod to a labeled node.
* Label another node and assign a pod to it.

---

### Label a node

Labeling nodes allows you, among other things, to constrain which nodes your Pod is eligible to be scheduled on. This concept is called ```Node affinity```.

There are two types of node affinity: ```requiredDuringSchedulingIgnoredDuringExecution``` and ```preferredDuringSchedulingIgnoredDuringExecution```. The first specifies that a condition must be met in order to apply scheduling onto a node, and in the latter the scheduler will try to enforce but will not guarantee.

```IgnoredDuringExecution``` means that if a Pod is already deployed on a node and some condition changes (e.g. label), the Pod won't be evicted from the node and will keep running.

---

Both master and worker nodes have labels already set.

```bash
$ student@master: k describe node master |grep -i label -A6
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=master
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
```

```bash
$ student@master: k describe node worker |grep -i label -A6
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=worker
                    kubernetes.io/os=linux
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
```

---

Let's manage ```two labels```, black and white.

Label the worker node as ```color=black```.

```bash
$ student@master: k label node worker color=black
node/worker labeled
```

```bash
$ student@master: k describe node worker |grep -i label -A6
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    color=black
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=worker
                    kubernetes.io/os=linux
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
```

---

### Assign a pod to a labeled node

First, create a simple deployment named black, with 3 replicas.

```bash
$ student@master: k create deploy black --image=nginx --replicas=3
deployment.apps/black created
```

Note in which node Pods are deployed on.

```bash
$ student@master: k get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
black-c8697748f-bxcsz   1/1     Running   0          4s    192.168.171.67   worker   <none>           <none>
black-c8697748f-fknfg   1/1     Running   0          4s    192.168.219.70   master   <none>           <none>
black-c8697748f-wh65v   1/1     Running   0          4s    192.168.171.66   worker   <none>           <none>
```

> _Output can differ, but it is highly probable that not all Pods are running on the same node (regardless of their labels)._

Now, force the Pods of this deployment to run on the ```worker``` node (labeled as ```color=black```).

To do it, delete the deploy, create a yaml template and add the appropriate node affinity.

```bash
$ student@master: k delete deploy black
deployment.apps "black" deleted
```

```bash
$ student@master: k create deploy black --image=nginx --replicas=3 --dry-run=client -o yaml > yaml/black.yaml
```

```yaml
# black.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null # Can be erased
  labels:
    app: black
  name: black
spec:
  replicas: 3
  selector:
    matchLabels:
      app: black
  strategy: {} # Can be erased
  template:
    metadata:
      creationTimestamp: null # Can be erased
      labels:
        app: black
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {} # Can be erased
status: {} # Can be erased
```

Add the node affinity section at the same level of ```containers```. We'll be using ```requiredDuringSchedulingIgnoredDuringExecution```.

```yaml
apiVersion: apps/v1
kind: Deployment
# Omitted
spec:
# Omitted
    spec:
      containers:
      - image: nginx
        name: nginx
      # New lines
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In # Values can be In, NotIn, Exists, DoesNotExist, Gt
                values:
                - black
```

All the Pods should be running on the worker node.

```bash
$ student@master: k get pods -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
black-cc6b698-cns4h   1/1     Running   0          5s    192.168.171.68   worker   <none>           <none>
black-cc6b698-gw7kt   1/1     Running   0          5s    192.168.171.73   worker   <none>           <none>
black-cc6b698-l76bk   1/1     Running   0          5s    192.168.171.69   worker   <none>           <none>
```

---

### Label another node and assign a pod to it

Create the same kind of deployment and force to run on the master node. In this case, there is a label which is only present in it (```node-role.kubernetes.io/master```).

Let's name it as white and add the node affinity spec.

```bash
$ student@master: k create deploy white --image=nginx --replicas=3 --dry-run=client -o yaml > yaml/white.yaml
```

It is enough to use the ```Exists``` operator because the master node has the key and not the value in this label.

```yaml
# white.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: white
  name: white
spec:
  replicas: 3
  selector:
    matchLabels:
      app: white
  template:
    metadata:
      labels:
        app: white
    spec:
      containers:
      - image: nginx
        name: nginx
      # New lines
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
```

```bash
$ student@master: k apply -f yaml/white.yaml
deployment.apps/white created
```

Now, all the white Pods should be running on the master node.

```bash
$ student@master: k get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
black-cc6b698-cns4h      1/1     Running   0          7m28s   192.168.171.68   worker   <none>           <none>
black-cc6b698-gw7kt      1/1     Running   0          7m28s   192.168.171.73   worker   <none>           <none>
black-cc6b698-l76bk      1/1     Running   0          7m28s   192.168.171.69   worker   <none>           <none>
white-67f7bf7fd4-g7rmg   1/1     Running   0          12s     192.168.219.68   master   <none>           <none>
white-67f7bf7fd4-jbhlb   1/1     Running   0          12s     192.168.219.69   master   <none>           <none>
white-67f7bf7fd4-nknlb   1/1     Running   0          12s     192.168.219.71   master   <none>           <none>
```

---

### Cleanup

```bash
k delete -f yaml/black.yaml
k delete -f yaml/white.yaml

deployment.apps "black" deleted
deployment.apps "white" deleted
```

```bash
# Remove the color label
$ k label node worker color-
node/worker labeled
```

---

## References

* [Assign Pods to Nodes (official site)](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
