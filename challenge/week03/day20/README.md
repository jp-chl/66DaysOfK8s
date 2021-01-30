# Day 20 of #66DaysOfK8s

_Last update: 2021-01-30_

---

Today, I have worked with Constraints for namespaces.
We'll be setting LimitRange to a namespace.

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

* Work with constraints for namespaces

---

### Connect to the master node

Connect to the worker node and create a namespace called ```restricted```.

```bash
student@master: ̃$ k create ns restricted
namespace/restricted created
```

Create a LimitRange object for resource limits setup.

```bash
student@worker:~$ vi restricted.yaml
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: restricted
spec:
  limits:
  - default:
      cpu: 1
      memory: 500Mi
    defaultRequest:
      cpu: 0.5
      memory: 100Mi
    type: Container
```

```bash
student@master:~$ k -n restricted create -f restricted.yaml
limitrange/restricted created
```

```bash
student@master:~$ k -n restricted get limitrange
NAME         CREATED AT
restricted   2021-01-30T21:44:13Z
```

---

Create a deployment in new namespace.

```bash
student@master:~$ kubectl -n restricted create deploy restricted-hog --image vish/stress
deployment.apps/restricted-hog created
```

```bash
student@master:~$ kubectl -n restricted get pod $(kubectl -n restricted get pods -o jsonpath='{.items[0].metadata.name}') -o yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 192.168.171.84/32
    cni.projectcalico.org/podIPs: 192.168.171.84/32
    kubernetes.io/limit-ranger: 'LimitRanger plugin set: cpu, memory request for container
      stress; cpu, memory limit for container stress'
  creationTimestamp: "2021-01-30T21:47:11Z"
# Output omitted
spec:
  containers:
  - image: vish/stress
    imagePullPolicy: Always
    name: stress
    resources:
      limits:
        cpu: "1"
        memory: 500Mi
      requests:
        cpu: 500m
        memory: 100Mi
    terminationMessagePath: /dev/termination-log
# Output omitted
```

```bash
student@master:~$ kubectl get deploy,rs,po -A
NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/hog                       1/1     1            1           6m3s
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           13d
kube-system   deployment.apps/coredns                   2/2     2            2           13d
restricted    deployment.apps/restricted-hog            1/1     1            1           3m23s
```

---

In contrast with the deployment with no constraints (check [last exercise link](../day19)), top command won't show a stress process due to the LimitRange.

```bash
student@master: ̃$ top

# Output omitted
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1210 root      20   0 1861416  97088  62768 S   2.0  1.3   2:17.84 kubelet
 3496 root      20   0 1578468  54284  36360 S   1.7  0.7   1:39.27 calico-node
# Output omitted
```

---

In the last exercise, a stress process was created like the following.

```bash
student@master: ̃$ kubectl create deployment hog --image vish/stress
deployment.apps/hog created
```

```bash
student@worker:~$ k get deploy hog -o yaml > hog.yaml
```

Set a stress deployment by adding the following to the yaml file.

```yaml
        resources:
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

And apply the configuration.

```bash
student@master: ̃$ kubectl delete deployment hog
deployment.apps "hog" deleted
```

```bash
student@master: ̃$ kubectl create -f hog.yaml
deployment.apps/hog created
```

Now you'll notice a stress process at the top.

```bash
student@master: ̃$ top

# Output omitted
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1018 root      20   0  958532 954464   3096 R 100.7 12.5  20:16.69 stress
# Output omitted
```

---

Create a new yaml manifest based on last stress deployment definition, but change namespace to ```restricted``` and comment selfLink line.

```bash
student@master: ̃$ cp hog.yaml hog2.yaml
```

```bash
student@master: ̃$ vi hog2.yaml
```

```yaml
  name: hog
  #namespace: default
  namespace: restricted
  resourceVersion: "172569"
  #selfLink: /apis/apps/v1/namespaces/default/deployments/hog
```

> _Per-deployment setting override the global namespace setting, so you'll see a stress process at the top (of the other node )._

```bash
student@master: ̃$ top

# Output omitted
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 2533 root      20   0  958532 954652   3180 S 100.0 12.5   4:09.65 stress
# Output omitted
```

---

## Cleanup

```bash
student@master:~$ k -n restricted delete deploy hog
deployment.apps "hog" deleted
```

```bash
student@master:~$ k delete deploy hog
deployment.apps "hog" deleted
```

---
## References

* [Limit Ranges (official site)](https://kubernetes.io/docs/concepts/policy/limit-range/)
