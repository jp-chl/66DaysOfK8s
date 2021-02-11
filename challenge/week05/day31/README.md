# Day 31 of #66DaysOfK8s

_Last update: 2021-02-10_

---
Today, I have practiced with faster ways to create pods, deployments and services using imperative commands.

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
* Set an alias for kubectl (```alias k=kubectl```)

---

## Tasks

* Create pod with specific image, add labels.
* Create a pod, exposing a container port.
* Expose an existent pod as a service.
* Create a deployment with selected image and replicas.

---

### Create pod with specific image, add labels

You can spawn a pod with:

```
kubectl run <pod name> --image=<container image>
```

For example, to start a pod named ```nginx``` with an image ```image:alpine```

```bash
$ student@master: kubectl run nginx --image=nginx:alpine
pod/nginx created
```

```bash
$ student@master: kubectl describe pod nginx
Name:         nginx
Namespace:    default
# Output omitted
Containers:
  nginx:
    Container ID:   docker://2745b8b7f12c04b915478c111d2e67f35cd7c435a462a39404a77206c7c5824a
    Image:          nginx:alpine
# Output omitted
  Normal  Started    5m6s  kubelet, worker    Started container nginx
```

```bash
# Shorter way to check right image
$ student@master: kubectl describe pod nginx |grep -i image
    Image:          nginx:alpine
    Image ID:       docker-pullable://nginx@sha256:c2ce58e024275728b00a554ac25628af25c54782865b3487b11c21cafb7fabda
  Normal  Pulled     4m50s  kubelet, worker    Container image "nginx:alpine" already present on machine
```

---

Now, create a pod called ```redis``` with image ```redis:alpine``` and labeling as "```dbtype=memory```".

```bash
$ student@master: k run redis --image=redis:alpine -l=dbtype=memory
pod/redis created
```

```bash
# Check correct labeling
$ student@master: k describe pod redis |grep -i label
Labels:       dbtype=memory
```

---

### Expose an existent pod as a service

Expose last created ```redis``` pod and expose it at port ```1234```.

```bash
$ student@master: k expose pod redis --port=1234 --name=redis-service
service/redis-service exposed
```

```bash
$ student@master: k describe svc redis-service|grep -i port
Port:              <unset>  1234/TCP
TargetPort:        1234/TCP
```

---

### Create a deployment with selected image and replicas

Create a deployment named ```httpd``` with image ```httpd``` with **two replicas**.

```bash
$ student@master: k create deploy httpd --image=httpd --replicas=2
deployment.apps/httpd created
```

```bash
$ student@master: k describe deploy httpd|grep -i image
    Image:        httpd
```

```bash
$ student@master: k describe deploy httpd|grep -i replica
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
# Output omitted
```

---

### Cleanup

```bash
k delete svc redis-service
k delete deploy httpd
k delete pod redis
k delete pod nginx

service "redis-service" deleted
deployment.apps "httpd" deleted
pod "redis" deleted
pod "nginx" deleted
```
