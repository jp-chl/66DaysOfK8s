# Day 2 of #66DaysOfK8s

_Last update: 2021-01-12_

---

Today, I've learned to create immutable pods. It allows us easy rollback, more reliability, better security, and always know the state of the pod.

> _Based on: https://itnext.io/cks-exam-series-3-immutable-pods-3812cf76cff4_

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

* All tests runs on minikube.
* All pods are deployed on default namespace

---

## Tasks:

1. Create _Pod_ ```holiday``` with two containers ```c1``` and ```c2``` of image ```bash:5.1.0```, ensure the containers keep running

2. Create _Deployment_ ```snow``` of image ```nginx:1.19.6``` with 3 replicas

3. Force container ```c2``` of _Pod_ ```holiday``` to run immutable: no files can be changed during runtime

4. Make sure the container of _Deployment_ ```snow``` will run immutable. Then make necessary paths writable for Nginx to work.

5. Verify everything

---

## Code

Create pod holiday:

```bash
kubectl run holiday --image=bash:5.1.0 --command -oyaml --dry-run -- sh -c 'sleep 1d' > ./holiday.yaml
```

Last command produces:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: holiday
  name: holiday
spec:
  replicas: 1
  selector:
    matchLabels:
      run: holiday
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: holiday
    spec:
      containers:
      - command:
        - sh
        - -c
        - sleep 1d
        image: bash:5.1.0
        name: holiday
        resources: {}
status: {}
```

---

Create snow deploy:

```bash
kubectl create deploy snow --image=nginx:1.19.6 -oyaml --dry-run > snow.yaml
```

Last command produces:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: snow
  name: snow
spec:
  replicas: 1 # Here, change the replicas to 3
  selector:
    matchLabels:
      app: snow
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: snow
    spec:
      containers:
      - image: nginx:1.19.6
        name: nginx
        resources: {}
status: {}
```

---

In ```holiday.yaml```, add a second container, add SecurityContext on the second one and change both container names to ```c1``` and ```c2```, respectively:

```yaml
apiVersion: apps/v1
kind: Deployment
# ...
spec:
# ...
    spec:
      containers:
# ...
      - command:
# ...
        name: c1 # former name: holiday
        resources: {}
# Adding a second container
      - command:
        - sh
        - -c
        - sleep 1d
        image: bash:5.1.0
        name: c2
        resources: {}
        securityContext:
          readOnlyRootFilesystem: true
        securityContext: # applies only to 2nd container
          readOnlyRootFilesystem: true # applies only to 2nd container
status: {}
```

---

Start holiday pod and check write permissions in both containers.

```bash
$ kubectl apply -f ./holiday.yaml
deployment.apps/holiday created
```

```bash
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
holiday-6769464945-2hst8   2/2     Running   0          35m
```

```bash
# container 1: "-c c1"
$ kubectl exec -ti $(kubectl get pods -l "run=holiday" -o jsonpath='{.items[0].metadata.name}') -c c1 -- touch /tmp/test
# It works, no problem
```

```bash
# container 2: "-c c2"
$ kubectl exec -ti $(kubectl get pods -l "run=holiday" -o jsonpath='{.items[0].metadata.name}') -c c2 -- touch /tmp/test
# It fails, as expected
touch: /tmp/test: Read-only file system
command terminated with exit code 1
```

---

Deploy snow:

```bash
$ kubectl apply -f ./snow.yaml
deployment.apps/snow created
```

```bash
$ kubectl get pods -l "app=snow"
NAME                    READY   STATUS    RESTARTS   AGE
snow-7fff94d5cb-dqmnp   1/1     Running   0          42s
snow-7fff94d5cb-htnfv   1/1     Running   0          42s
snow-7fff94d5cb-sxvzs   1/1     Running   0          42s
```

```bash
$ kubectl get deploy snow
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
snow   3/3     3            3           105s
```

```bash
# First pod: items[0]
$ kubectl exec -ti $(kubectl get pods -l "app=snow" -o jsonpath='{.items[0].metadata.name}') -- touch /tmp/test
# It works, no problem
```

> _The same result is expected on the other two pods ;-)_

---

In snow deployment (```snow.yaml```), add volume and volume mounts:

* /var/cache/nginx

* /var/run

```yaml
apiVersion: apps/v1
kind: Deployment
# ...
spec:
# ...
    spec:
      containers:
      - image: nginx:1.19.6
        name: nginx
        resources: {}
        # new lines start here
        securityContext:
          readOnlyRootFilesystem: true
        volumeMounts:
        - name: write1
          mountPath: /var/cache/nginx
        - name: write2
          mountPath: /var/run
      volumes:
      - name: write1
        emptyDir: {}
      - name: write2
        emptyDir: {}
      # new lines end here
status: {}
```

Deploy updated snow yaml:

```bash
$ kubectl apply -f ./snow.yaml
deployment.apps/snow configured
```

```bash
$ kubectl get pods -l "app=snow"
NAME                    READY   STATUS    RESTARTS   AGE
snow-575cd78c85-drxr7   1/1     Running   0          111s
snow-575cd78c85-fstdb   1/1     Running   0          108s
snow-575cd78c85-ghwfs   1/1     Running   0          105s
```

```bash
# First pod: items[0]
$ kubectl exec -ti $(kubectl get pods -l "app=snow" -o jsonpath='{.items[0].metadata.name}') -- touch /tmp/test
# It fails, as expected
touch: cannot touch '/tmp/test': Read-only file system
command terminated with exit code 1
```

```bash
# First pod: items[0]
# Attempt to write to "/var/cache/nginx" directory
$ kubectl exec -ti $(kubectl get pods -l "app=snow" -o jsonpath='{.items[0].metadata.name}') -- touch /var/cache/nginx/test
# It works, as expected
```

```bash
# First pod: items[0]
# Attempt to write to "/var/run" directory
$ kubectl exec -ti $(kubectl get pods -l "app=snow" -o jsonpath='{.items[0].metadata.name}') -- touch /var/run/test
# It works, as expected
```

The same result is expected on the other two pods ;-)

---

## Cleanup

```bash
$ kubectl delete -f ./holiday.yaml
deployment.apps "holiday" deleted
```

```bash
$ kubectl delete -f ./snow.yaml
deployment.apps "snow" deleted
```

---

# Demo

[![asciicast](https://asciinema.org/a/3wRUO190WXsv7a9TDbvXqsyeA.svg)](https://asciinema.org/a/3wRUO190WXsv7a9TDbvXqsyeA)
