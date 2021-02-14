# Day 34 of #66DaysOfK8s

_Last update: 2021-02-13_

---
Today, I have worked with node selector label to assign a Pods.

#kubernetes #learning #K8s #66DaysChallenge

---

## TL;DR



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

* Create a Pod, via deployment, and set a node selector to it.
* Tag a node with the proper label to enable Pod deployment.
* Untag the node.

---

### Create a Pod, via deployment, and set a node selector to it

Here's a simple nginx Pod with a specific node selector (```tier: web```).

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  labels:
    app: my-nginx
  namespace: default
spec:
  selector:
    matchLabels:
      app: my-nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
      nodeSelector:
        tier: web # Unless a node has this label, the Pod won't run
```

```bash
$ kubectl get nodes --show-labels
NAME       STATUS   ROLES    AGE   VERSION   LABELS
minikube   Ready    master   61m   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=minikube,kubernetes.io/os=linux,minikube.k8s.io/commit=0c5e9de4ca6f9c55147ae7f90af97eff5befef5f,minikube.k8s.io/name=minikube,minikube.k8s.io/updated_at=2021_02_13T22_09_56_0700,minikube.k8s.io/version=v1.13.0,node-role.kubernetes.io/master=
```

```bash
$ kubectl apply -f yaml/.
deployment.apps/my-nginx created
```

The pod will be in Pending status until a node has a label ```tier=web```.

```bash
$ kubectl get po,deploy
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-57848877bb-6vpmr   0/1     Pending   0          6s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   0/1     1            0           6s
```

---

### Tag a node with the proper label to enable Pod deployment

```bash
$ kubectl label node minikube tier=web
node/minikube labeled
```

Now the Pod must run.

```bash
$ kubectl get po,deploy
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-57848877bb-6vpmr   1/1     Running   0          3m27s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   1/1     1            1           3m27s
```

---

### Untag the node

Any Pod that is already running will continue to do so even though the node, in which is assigned, lose the label.

```bash
kubectl label node minikube tier-
node/minikube labeled
```

```bash
kubectl get po,deploy
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-57848877bb-6vpmr   1/1     Running   0          5m48s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   1/1     1            1           5m48s
```


However, if we scale up the deployment, new Pods will be in Pending state.

```bash
$ kubectl scale deployments/my-nginx --replicas=2
deployment.apps/my-nginx scaled
```

```bash
kubectl get po,deploy
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-57848877bb-6vpmr   1/1     Running   0          12m
pod/my-nginx-57848877bb-v9k5f   0/1     Pending   0          8s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   1/2     2            1           12m
```

---

### Cleanup

```bash
$ kubectl delete -f yaml/.
deployment.apps/my-nginx deleted
```

---

## References

* [Assigning Pods to Nodes (official site)](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

* [Labels and Selectors (official site)](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

---

# Demo

[![asciicast](https://asciinema.org/a/FzeiHuu8naKodNlbCYuzxa0as.svg)](https://asciinema.org/a/FzeiHuu8naKodNlbCYuzxa0as)
