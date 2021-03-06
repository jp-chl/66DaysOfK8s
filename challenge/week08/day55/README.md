# Day 55 of #66DaysOfK8s

_Last update: 2021-03-06_

---
Today I have worked in part 2 of Resource quota exercises.

#kubernetes #learning #K8s #66DaysChallenge

---

## Setup

* Minikube, by default, gives you admin access to all resources. 
* Set an alias for kubectl (```alias k=kubectl```) plus execute the following command: ```complete -F __start_kubectl k```.
* All tests run on ```quota-test``` namespace.

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Tasks

* Exercise pod quota
* Exercise secret quota
* Other count quota examples

---

## Exercise pod quota

```bash
$ k create ns quota-test
namespace/quota-test created
```

Let's define a Pod quota of 2.

```bash
$ k -n quota-test create resourcequota test-pod-quota --hard=pods="2" --dry-run -o yaml
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: test-pod-quota
  namespace: quota-test
spec:
  hard:
    pods: "2"
status: {}
```

```bash
$ k -n quota-test create resourcequota test-pod-quota --hard=pods="2"
resourcequota/test-pod-quota created
```

```bash
$ k -n quota-test describe quota/test-pod-quota
Name:       test-pod-quota
Namespace:  quota-test
Resource    Used  Hard
--------    ----  ----
pods        0     2
```

So, two Pods will be created effortless and the third one must raise an error.

```bash
$ k -n quota-test run pod1 --generator=run-pod/v1 --image=busybox --command sleep 3600 --restart=Never
pod/pod1 created
```

```bash
$ k -n quota-test run pod2 --generator=run-pod/v1 --image=busybox --command sleep 3600 --restart=Never
pod/pod2 created
```

```bash
$ k -n quota-test describe quota/test-pod-quota
Name:       test-pod-quota
Namespace:  quota-test
Resource    Used  Hard
--------    ----  ----
pods        2     2
```

```bash
$ k -n quota-test run pod3 --generator=run-pod/v1 --image=busybox --command sleep 3600 --restart=Never
Error from server (Forbidden): pods "pod3" is forbidden: exceeded quota: test-pod-quota, requested: pods=1, used: pods=2, limited: pods=2
```

---

Cleanup:

```bash
$ k delete ns quota-test --wait=false
namespace "quota-test" deleted
```

---

## Exercise secret quota

```bash
$ k create ns quota-test
namespace/quota-test created
```

Let's define a Secret quota of 2.

```bash
$ k -n quota-test create resourcequota test-secret-quota --hard=secrets="2" --dry-run -o yaml
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: test-secret-quota
  namespace: quota-test
spec:
  hard:
    secrets: "2"
status: {}
```

```bash
$ k -n quota-test create resourcequota test-secret-quota --hard=secrets="2"
resourcequota/test-secret-quota created
```

```bash
$ k -n quota-test describe quota/test-secret-quota
Name:       test-secret-quota
Namespace:  quota-test
Resource    Used  Hard
--------    ----  ----
secrets     1     2
```

A secret (default) per namespace is always created.

```bash
$ k -n quota-test get secrets
NAME                  TYPE                                  DATA   AGE
default-token-xplsq   kubernetes.io/service-account-token   3      6m6s
```

```bash
$ k -n quota-test create secret generic my-secret --from-literal=secretKey=secretValue
secret/my-secret created
```

```bash
$ k -n quota-test describe quota/test-secret-quota
Name:       test-secret-quota
Namespace:  quota-test
Resource    Used  Hard
--------    ----  ----
secrets     2     2
```

Now, a new secret creation is going to raise an error.

```bash
$ k -n quota-test create secret generic my-secret-2 --from-literal=secretKey=secretValue
Error from server (Forbidden): secrets "my-secret-2" is forbidden: exceeded quota: test-secret-quota, requested: secrets=1, used: secrets=2, limited: secrets=2
```

---

Cleanup:

```bash
$ k delete ns quota-test --wait=false
namespace "quota-test" deleted
```

---

## Other count quota examples

According to official site:

_Here is an example set of resources users may want to put under object count quota:_

```
count/persistentvolumeclaims
count/services
count/secrets
count/configmaps
count/replicationcontrollers
count/deployments.apps
count/replicasets.apps
count/statefulsets.apps
count/jobs.batch
count/cronjobs.batch
```

---

## References

* [Resource Quotas (official site)](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
