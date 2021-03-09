# Day 57 of #66DaysOfK8s

_Last update: 2021-03-08_

---
Today I have worked in the 2nd part of Deployment management (Rolling Update).

#kubernetes #learning #K8s #66DaysChallenge

---

## Setup

* Minikube, by default, gives you admin access to all resources. 
* Set an alias for kubectl (```alias k=kubectl```) plus execute the following command: ```complete -F __start_kubectl k```.
* All tests run on ```quota-test``` namespace.
* _Optional_: stern (```brew install stern```)

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0
* stern: v1.11.0

---

## Tasks

* Create a deployment with several replicas
* Generate replica sets by changing deployment container associated image
* Explore rollout command

---

## Create a deployment with several replicas

Refer to [part 1](../../week08/day56/) for all the details.

```bash
$ k create deploy myapp --image=quay.io/rhdevelopers/quarkus-demo:v1
deployment.apps/myapp created
```

```bash
$ k scale deploy myapp --replicas=3
deployment.apps/myapp scaled
```

---

## Generate replica sets by changing deployment container associated image

Edit deployment and change its image.

```bash
$ k edit deploy myapp
```

From ```quarkus-demo:v1```, ```myboot:v1```.

```yaml
# Output omitted
    spec:
      containers:
      - image: quay.io/rhdevelopers/myboot:v1
# Output omitted
```

```bash
deployment.apps/myapp edited
```

---

Let's add a new replica set. Edit deployment and change its image.

```bash
$ k edit deploy myapp
```

From ```myboot:v1```, ```myboot:v2```.

```yaml
# Output omitted
    spec:
      containers:
      - image: quay.io/rhdevelopers/myboot:v2
# Output omitted
```

---

## Explore rollout command

There should be 3 deployment revisions.

```bash
$ k rollout history deploy myapp
deployment.apps/myapp
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

If we roll back 1st version (quarkus-demo:v1), no new replica set will be created but re-used.

```bash
$ k rollout undo deploy myapp --to-revision=1
deployment.apps/myapp rolled back
```

```bash
$ k get all
NAME                         READY   STATUS        RESTARTS   AGE
pod/myapp-7457bf76dd-5zqkk   1/1     Terminating   0          10m
pod/myapp-7457bf76dd-x5cn6   1/1     Terminating   0          10m
pod/myapp-7457bf76dd-x8gst   1/1     Terminating   0          10m
pod/myapp-784dbf86d9-65mx5   1/1     Running       0          6s
pod/myapp-784dbf86d9-99knt   1/1     Running       0          9s
pod/myapp-784dbf86d9-j5wk2   1/1     Running       0          4s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   18m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp   3/3     3            3           18m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-7457bf76dd   0         0         0       10m
replicaset.apps/myapp-784dbf86d9   3         3         3       18m
replicaset.apps/myapp-78b95578f    0         0         0       13m
```

---

Let's check what happens if we roll back to the 2nd version (myboot:v1).

```bash
$ k rollout undo deploy myapp --to-revision=2
deployment.apps/myapp rolled back
```

```bash
$ k get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/myapp-78b95578f-pd9jk   1/1     Running   0          7s
pod/myapp-78b95578f-s6dhs   1/1     Running   0          10s
pod/myapp-78b95578f-td29n   1/1     Running   0          13s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   20m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp   3/3     3            3           20m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-7457bf76dd   0         0         0       12m
replicaset.apps/myapp-784dbf86d9   0         0         0       20m
replicaset.apps/myapp-78b95578f    3         3         3       15m
```

---

## Cleanup

```bash
$ k delete all --all
# Output omitted 
pod "myapp-...." deleted
service "kubernetes" deleted
service "myapp" deleted
deployment.apps "myapp" deleted
replicaset.apps "myapp-7457bf76dd" deleted

replicaset.apps "myapp-78b95578f" deleted
```

---

## References

* [Deployment (official site)](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [Article by Keilan Jackson (Bluematador Blog)](https://www.bluematador.com/blog/kubernetes-deployments-rolling-update-configuration)
