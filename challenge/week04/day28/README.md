# Day 28 of #66DaysOfK8s

_Last update: 2021-02-07_

---

Today, I have worked with an introduction to Daemon sets.

#kubernetes #learning #K8s #66DaysChallenge

---

## TL;DR

This is a practical exercise as a first approach to a DaemonSet controller in K8s. They are used to creates objects in every node (normally one pod, for example to handle logging).

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client

---

## Tasks

* Create a simple DaemonSet and check its state in every node.

---

### Create a simple DaemonSet and check its state in every node

We can copy a ReplicaSet template like the one available on this [link](https://github.com/jp-chl/66DaysOfK8s/blob/master/challenge/week04/day26/yaml/rs.yaml), and changing it to a DaemonSet manifest.

```yaml
# original ReplicaSet
apiVersion: apps/v1
kind: ReplicaSet # <-- Change to DaemonSet
metadata:
  name: my-rs # <-- Change
spec:
  replicas: 2 # <-- Remove line
  selector:
    matchLabels: 
      system: MyReplica # <-- Change
  template:
    metadata:
      labels:
        system: MyReplica # <-- Change
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```yaml
# ds.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-ds
spec:
  #replicas: 2
  selector:
    matchLabels: 
      system: MyDaemonSet
  template:
    metadata:
      labels:
        system: MyDaemonSet
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```bash
student@master:~$ kubectl apply -f yaml/ds.yaml
daemonset.apps/my-ds created
```

---

Now, every node has its one daemon set.

```bash
student@master:~$ kubectl get ds
NAME    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
my-ds   2         2         2       2            2           <none>          17s
```

```bash
student@master:~$ kubectl get pods -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
my-ds-tglw8   1/1     Running   0          30s   192.168.219.112   master   <none>           <none>
my-ds-zmc2p   1/1     Running   0          30s   192.168.171.95    worker   <none>           <none>
```

---

### Cleanup

```bash
student@master:~$ kubectl delete -f yaml/ds.yaml
```

---

## References

* [DaemonSet (official github)](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
