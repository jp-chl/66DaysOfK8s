# Day 56 of #66DaysOfK8s

_Last update: 2021-03-07_

---
Today I have worked in the 1st part of Deployment management.

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

---

## Create a deployment with several replicas

We'll be using 3 simple container images from RedHat. Each of them expose an endpoint which displays a message with hostname (pod name) and an invocation count (```quay.io/rhdevelopers/quarkus-demo:v1```, ```quay.io/rhdevelopers/myboot:v1``` and ```quay.io/rhdevelopers/myboot:v2```).

```bash
$ k create deploy myapp --image=quay.io/rhdevelopers/quarkus-demo:v1
deployment.apps/myapp created
```

```
$ k get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/myapp-784dbf86d9-w8g8k   1/1     Running   0          39s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   45s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp   1/1     1            1           39s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-784dbf86d9   1         1         1       39s
```

```bash
$ k scale deploy myapp --replicas=3
deployment.apps/myapp scaled
```

```bash
$ k get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/myapp-784dbf86d9-s7xl6   1/1     Running   0          48s
pod/myapp-784dbf86d9-vl6fv   1/1     Running   0          48s
pod/myapp-784dbf86d9-w8g8k   1/1     Running   0          112s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   118s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp   3/3     3            3           112s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-784dbf86d9   3         3         3       112s
```

```bash
$ stern myapp
```

```log
+ myapp-784dbf86d9-w8g8k › quarkus-demo
+ myapp-784dbf86d9-vl6fv › quarkus-demo
+ myapp-784dbf86d9-s7xl6 › quarkus-demo
myapp-784dbf86d9-vl6fv quarkus-demo __  ____  __  _____   ___  __ ____  ______
myapp-784dbf86d9-vl6fv quarkus-demo  --/ __ \/ / / / _ | / _ \/ //_/ / / / __/
myapp-784dbf86d9-vl6fv quarkus-demo  -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \
myapp-784dbf86d9-vl6fv quarkus-demo --\___\_\____/_/ |_/_/|_/_/|_|\____/___/
myapp-784dbf86d9-vl6fv quarkus-demo 2021-03-09 02:59:58,176 INFO  [io.quarkus] (main) quarkus-demo 2.0.0 (powered by Quarkus 1.3.2.Final) started in 0.075s. Listening on: http://0.0.0.0:8080
myapp-784dbf86d9-vl6fv quarkus-demo 2021-03-09 02:59:58,177 INFO  [io.quarkus] (main) Profile prod activated.
myapp-784dbf86d9-vl6fv quarkus-demo 2021-03-09 02:59:58,177 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
myapp-784dbf86d9-s7xl6 quarkus-demo __  ____  __  _____   ___  __ ____  ______
myapp-784dbf86d9-s7xl6 quarkus-demo  --/ __ \/ / / / _ | / _ \/ //_/ / / / __/
myapp-784dbf86d9-s7xl6 quarkus-demo  -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \
myapp-784dbf86d9-s7xl6 quarkus-demo --\___\_\____/_/ |_/_/|_/_/|_|\____/___/
myapp-784dbf86d9-s7xl6 quarkus-demo 2021-03-09 02:59:58,393 INFO  [io.quarkus] (main) quarkus-demo 2.0.0 (powered by Quarkus 1.3.2.Final) started in 0.045s. Listening on: http://0.0.0.0:8080
myapp-784dbf86d9-s7xl6 quarkus-demo 2021-03-09 02:59:58,393 INFO  [io.quarkus] (main) Profile prod activated.
myapp-784dbf86d9-s7xl6 quarkus-demo 2021-03-09 02:59:58,394 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
myapp-784dbf86d9-w8g8k quarkus-demo __  ____  __  _____   ___  __ ____  ______
myapp-784dbf86d9-w8g8k quarkus-demo  --/ __ \/ / / / _ | / _ \/ //_/ / / / __/
myapp-784dbf86d9-w8g8k quarkus-demo  -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \
myapp-784dbf86d9-w8g8k quarkus-demo --\___\_\____/_/ |_/_/|_/_/|_|\____/___/
myapp-784dbf86d9-w8g8k quarkus-demo 2021-03-09 02:58:54,311 INFO  [io.quarkus] (main) quarkus-demo 2.0.0 (powered by Quarkus 1.3.2.Final) started in 0.043s. Listening on: http://0.0.0.0:8080
myapp-784dbf86d9-w8g8k quarkus-demo 2021-03-09 02:58:54,312 INFO  [io.quarkus] (main) Profile prod activated.
myapp-784dbf86d9-w8g8k quarkus-demo 2021-03-09 02:58:54,312 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
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

A new replica set is created.

```bash
$ k get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/myapp-78b95578f-hjp6n   1/1     Running   0          13s
pod/myapp-78b95578f-n4gw5   1/1     Running   0          20s
pod/myapp-78b95578f-nqzwz   1/1     Running   0          17s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5m26s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp   3/3     3            3           5m20s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-784dbf86d9   0         0         0       5m20s
replicaset.apps/myapp-78b95578f    3         3         3       20s
```

```bash
$ stern myapp
```

```log
...

myapp-78b95578f-n4gw5 quarkus-demo 2021-03-09 03:04:39.147  INFO 7 --- [           main] com.burrsutter.HellobootApplication      : Started HellobootApplication in 41.918 seconds (JVM running for 44.338)

...

myapp-78b95578f-nqzwz quarkus-demo 2021-03-09 03:04:48.893  INFO 6 --- [           main] com.burrsutter.HellobootApplication      : Started HellobootApplication in 41.311 seconds (JVM running for 50.652)

...

myapp-78b95578f-hjp6n quarkus-demo 2021-03-09 03:04:51.170  INFO 6 --- [           main] com.burrsutter.HellobootApplication      : Started HellobootApplication in 38.71 seconds (JVM running for 45.441)

...
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

```bash
deployment.apps/myapp edited
```


```bash
$ k get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/myapp-7457bf76dd-5zqkk   1/1     Running   0          80s
pod/myapp-7457bf76dd-x5cn6   1/1     Running   0          75s
pod/myapp-7457bf76dd-x8gst   1/1     Running   0          83s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   9m43s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp   3/3     3            3           9m37s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-7457bf76dd   3         3         3       83s
replicaset.apps/myapp-784dbf86d9   0         0         0       9m37s
replicaset.apps/myapp-78b95578f    0         0         0       4m37s
```

```bash
$ stern myapp
```

```log
...

myapp-7457bf76dd-x8gst quarkus-demo 2021-03-09 03:07:51.137  INFO 7 --- [           main] com.burrsutter.HellobootApplication      : Started HellobootApplication in 41.144 seconds (JVM running for 42.884)

...

myapp-7457bf76dd-5zqkk quarkus-demo 2021-03-09 03:07:59.090  INFO 7 --- [           main] com.burrsutter.HellobootApplication      : Started HellobootApplication in 42.283 seconds (JVM running for 46.439)

...

myapp-7457bf76dd-x5cn6 quarkus-demo 2021-03-09 03:08:02.852  INFO 6 --- [           main] com.burrsutter.HellobootApplication      : Started HellobootApplication in 39.167 seconds (JVM running for 45.242)

...
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

* [Part 2](../../week09/day57)
* [Deployment (official site)](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
