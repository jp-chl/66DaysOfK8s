# Day 54 of #66DaysOfK8s

_Last update: 2021-03-05_

---
Today I have worked with Resource quota, which is useful to restrict resources in a namespace.

_Based on an [Opensource.com article by Mike Calizo](https://opensource.com/article/20/12/kubernetes-resource-quotas)_.

#kubernetes #learning #K8s #66DaysChallenge

---

## Setup

* Minikube, by default, gives you admin access to all resources. 
* Set an alias for kubectl (```alias k=kubectl```) plus execute the following command: ```complete -F __start_kubectl k```.

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Tasks

* Create a namespace and associate a ResourceQuota to it
* Create pods until the quota is exceeded

---

## Create a namespace and associate a ResourceQuota to it

_"A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per namespace. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that namespace."_  -- (official site)


```bash
$ k create ns quota-test
namespace/quota-test created
```

```bash
$ k -n quota-test create resourcequota test-cpu-quota --hard=requests.cpu="100m",limits.cpu="200m" --dry-run -o yaml
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: test-cpu-quota
  namespace: quota-test
spec:
  hard:
    limits.cpu: 200m
    requests.cpu: 100m
status: {}
```

```bash
$ k -n quota-test create resourcequota test-cpu-quota --hard=requests.cpu="100m",limits.cpu="200m"
resourcequota/test-cpu-quota created
```

```bash
$ k -n quota-test describe quota/test-cpu-quota
Name:         test-cpu-quota
Namespace:    quota-test
Resource      Used  Hard
--------      ----  ----
limits.cpu    0     200m
requests.cpu  0     100m
```

---

## Create pods until the quota is exceeded

Let's create Pods consuming 100m each. Since namespace quota is 200m, a third Pod won't be spawned.

All Pods will have the same configuration like the following:

```bash
$ k run pod1 --generator=run-pod/v1 --image=busybox --requests=cpu=50m --limits=cpu=100m --command sleep 3600 --restart=Never --dry-run -o yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - command:
    - sleep
    - "3600"
    image: busybox
    name: pod1
    resources:
      limits:
        cpu: 100m
      requests:
        cpu: 50m
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create Pods in ```quota-test``` namespace:

```bash
$ k run pod1 --generator=run-pod/v1 --image=busybox --requests=cpu=50m --limits=cpu=100m --command sleep 3600 --restart=Never --dry-run -o yaml | k -n quota-test apply -f -
pod/pod1 created
```

```bash
# "quota" and "resourcequota" are aliases
k -n quota-test get quota/test-cpu-quota
NAME             AGE     REQUEST                  LIMIT
test-cpu-quota   3m52s   requests.cpu: 50m/100m   limits.cpu: 100m/200m
```

Another Pod:

```bash
$ k run pod2 --generator=run-pod/v1 --image=busybox --requests=cpu=50m --limits=cpu=100m --command sleep 3600 --restart=Never --dry-run -o yaml | k -n quota-test apply -f -
pod/pod2 created
```

```bash
$ k -n quota-test describe quota/test-cpu-quota
Name:         test-cpu-quota
Namespace:    quota-test
Resource      Used  Hard
--------      ----  ----
limits.cpu    200m  200m
requests.cpu  100m  100m
```

Finally, a third Pod will raise an error.

```bash
$ k run pod3 --generator=run-pod/v1 --image=busybox --requests=cpu=50m --limits=cpu=100m --command sleep 3600 --restart=Never --dry-run -o yaml | k -n quota-test apply -f -
Error from server (Forbidden): error when creating "STDIN": pods "pod3" is forbidden: exceeded quota: test-cpu-quota, requested: limits.cpu=100m,requests.cpu=50m, used: limits.cpu=200m,requests.cpu=100m, limited: limits.cpu=200m,requests.cpu=100m
```

---

## Cleanup

```bash
$ k delete ns quota-test --wait=false
namespace "quota-test" deleted
```

---

## References

* [Resource Quotas (official site)](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
