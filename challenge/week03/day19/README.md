# Day 19 of #66DaysOfK8s

_Last update: 2021-01-29_

---

Today, I have worked with CPU and memory constraints.
A stress container is used to try different configurations faster.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client

---

## Setup

* K8s cluster already created in GCP from scratch. Check the instructions in this [link](../../week01/day5/README.md).
* Set an alias for kubectl (```alias k=kubectl```)

---

## Tasks

* Work with memory and cpu constraints

---

### Connect to the master node

Connect to the worker node and create a deploy with a ```vish/stress```image.

```bash
student@master: ̃$ kubectl create deployment hog --image vish/stress
deployment.apps/hog created
```

```bash
student@worker:~$ k get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
hog    1/1     1            1           4s
```

```bash
student@worker:~$ k describe deploy hog
# Output omitted
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  63s   deployment-controller  Scaled up replica set hog-dc4488fc7 to 1
```

```bash
student@worker:~$ k logs hog-dc4488fc7-62l2m
I0130 02:18:21.652706       1 main.go:26] Allocating "0" memory, in "4Ki" chunks, with a 1ms sleep between allocations
I0130 02:18:21.652773       1 main.go:29] Allocated "0" memory
```

---

Save deployment as a yaml file and edit resources section with the next yaml extract.

```bash
student@worker:~$ k get deploy hog -o yaml > hog.yaml
```

```bash
student@worker:~$ vi hog.yaml
```

```yaml
         resources:
           limits:
             memory: "4Gi"
           requests:
            memory: "2500Mi"
```

Replace deployment configuration.

```bash
student@worker:~$ k replace -f hog.yaml
deployment.apps/hog replaced
```

```bash
student@worker:~$ kubectl get pods
NAME                   READY   STATUS        RESTARTS   AGE
hog-775c7c858f-r8rnw   1/1     Running       0          5s
hog-dc4488fc7-62l2m    0/1     Terminating   0          11m
```

```bash
student@worker:~$ kubectl get pods
NAME                   READY   STATUS        RESTARTS   AGE
hog-775c7c858f-r8rnw   1/1     Running       0          5s
```

```bash
student@worker:~$ k logs hog-775c7c858f-r8rnw
I0130 02:29:53.108798       1 main.go:26] Allocating "0" memory, in "4Ki" chunks, with a 1ms sleep between allocations
I0130 02:29:53.108868       1 main.go:29] Allocated "0" memory
```

---

Now edit yaml file again and add args to the container.

```bash
student@worker:~$ vi hog.yaml
```

```yaml
        resources:
          limits:
            cpu: "1" # new line
            memory: "4Gi"
          requests:
            cpu: "1" # new line
            memory: 2500Mi
        args:
        - -cpus
        - "2"
        - -mem-total
        - "950Mi"
        - -mem-alloc-size - "100Mi" # error
        - -mem-alloc-sleep - "1s" # error
```

Apply configuration.

```bash
student@master: ̃$ kubectl delete deployment hog
deployment.apps "hog" deleted
```

```bash
student@master: ̃$ kubectl create -f hog.yaml
deployment.apps/hog created
```

---

As you will notice pod does not start. And top shows no increase in memory usage.


```bash
kubectl get pods
NAME                   READY   STATUS        RESTARTS   AGE
hog-775c7c858f-r8rnw   0/1     Terminating   0          4m48s
```

```bash
student@worker:~$ top
top - 02:37:10 up 45 min,  3 users,  load average: 0.69, 0.31, 0.18
Tasks: 158 total,   1 running,  94 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.5 us,  2.5 sy,  0.0 ni, 94.8 id,  0.0 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem :  7636432 total,  6038504 free,   404124 used,  1193804 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  6994256 avail Mem
```


```bash
kubectl get pods
NAME                   READY   STATUS             RESTARTS   AGE
hog-79c46c9c96-7bvmk   0/1     CrashLoopBackOff   4          2m55s
```

---

It could be an error with args parameters in the yaml format.

```bash
student@worker:~$ k logs hog-79c46c9c96-7bvmk
flag provided but not defined: -mem-alloc-size - "100Mi"
Usage of /stress:
  -alsologtostderr
    	log to standard error as well as files
```

Fix it and try again.

```yaml
          limits:
            cpu: "1"
            memory: "4Gi"
          requests:
            cpu: "1"
            memory: 2500Mi
        args:
        - -cpus
        - "2"
        - -mem-total
        - "950Mi"
        - -mem-alloc-size # last error line
        - "100Mi" # new line
        - -mem-alloc-sleep # last error line
        - "1s" # new line
```

```bash
student@master: ̃$ kubectl delete deployment hog
deployment.apps "hog" deleted
```

```bash
student@master: ̃$ kubectl create -f hog.yaml
deployment.apps/hog created
```

Now it's working.

```bash
kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
hog-6f59cb5744-zddgh   1/1     Running   0          10
```

```bash
student@worker:~$ k logs hog-6f59cb5744-zddgh
I0130 02:40:58.027755       1 main.go:26] Allocating "950Mi" memory, in "100Mi" chunks, with a 1s sleep between allocations
I0130 02:40:58.027817       1 main.go:39] Spawning a thread to consume CPU
I0130 02:40:58.027833       1 main.go:39] Spawning a thread to consume CPU
I0130 02:41:13.277798       1 main.go:29] Allocated "950Mi" memory
```

```bash
# 1359856 used
student@worker:~$ top
top - 02:42:13 up 50 min,  3 users,  load average: 2.18, 0.84, 0.40
# Output ommited
KiB Mem :  7636432 total,  5081108 free,  1359856 used,  1195468 buff/cache
```

```bash
# 1360044 used (increase)
student@worker:~$ top
top - 02:43:32 up 51 min,  3 users,  load average: 2.72, 1.22, 0.56
# Output ommited
KiB Mem :  7636432 total,  5080664 free,  1360044 used,  1195724 buff/cache
```

---
## Cleanup

```bash
student@master:~$ k delete deploy hog
deployment.apps "hog" deleted
```

---
## References

* [Upgrade K8s cluster (official site)](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
