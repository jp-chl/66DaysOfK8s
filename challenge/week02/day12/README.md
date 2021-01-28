# Day 12 of #66DaysOfK8s

_Last update: 2021-01-22_

---

Today, I have worked in part 4 of a series of lessons in order to review the Kubernetes Architecture.
On this 4th day, a focus is on Pods and specifically on Init containers.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Setup

* All tests run on minikube.
* All pods and services are deployed on default namespace

---

## Tasks

* Understand Init containers
* Run an init containers example

---

### Init containers

A pod can have many containers running applications in it, but it can run also one or many init containers (tag "```initContainers```" within Pod specification).

A pod resources and limits are handled differently for them. They also don't support, for instance, [livenessProbe and readinessProbe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

[Kubelet](https://github.com/jp-chl/66DaysOfK8s/tree/master/challenge/week02/day10) starts the application containers only when all of the init containers have completed their tasks (_"run to completion"_). After that, the app containers can start in parallel.

Init containers can run with a different setup of the pod filesystem, and can access to [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) that app containers can't.

Any change to the init container image restarts the Pod.

---

**Init containers example**

A pod will run two init containers and one app container. The init containers will wait for some services to be available.

After applying the first yaml, the init containers will keep waiting until the second yaml has been applied (services creation). In the kubectl output, the status will show "```Init:0/2```"

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

```yaml
#services.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

```bash
$ kubectl apply -f pod.yaml
pod/myapp-pod created
```

```bash
kubectl get pods
NAME        READY   STATUS     RESTARTS   AGE
myapp-pod   0/1     Init:0/2   0          4s
```

```bash
$ kubectl exec -ti myapp-pod -c init-myservice -- cat /var/run/secrets/kubernetes.io/serviceaccount/namespace
default
```

```bash
$ grep nslookup pod.yaml
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

```bash
$ kubectl apply -f services.yaml
service/myservice created
service/mydb created
```

**Demo**:

[![asciicast](https://asciinema.org/a/FJtn3ju5OewbuajJrIlx7AZUu.svg)](https://asciinema.org/a/FJtn3ju5OewbuajJrIlx7AZUu)

---

## References

* [Part 5: Pods](../day13)

* [Pods (official site)](https://kubernetes.io/docs/concepts/workloads/pods/)

* [Init containers (official site)](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

* [Init container build pattern (Red Hat blog articles)](https://developers.redhat.com/blog/2019/04/01/init-container-build-pattern-knative-build-with-plain-old-kubernetes-deployment/)
