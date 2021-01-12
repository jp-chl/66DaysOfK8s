# Day 2 of #66DaysOfK8s

_Last update: 2021-01-12_

---

Today, I've learned to create immutable pods. It allows us easy rollback, more reliability, better security, and always know the state of the pod.

> _Based on: https://itnext.io/cks-exam-series-3-immutable-pods-3812cf76cff4_

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* minikube version: v1.13.0
* kubectl Client Version: v1.17.4
* kubectl Server Version: v1.19.0

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

In holiday.yaml, add SecurityContext on container level:

```yaml
apiVersion: apps/v1
# ...
spec:
# ...
    spec:
      containers:
# ...
      - command:
# ...
        name: c2
        resources: {}
        securityContext: # new line
          readOnlyRootFilesystem: true # new line
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
$ kubectl exec -ti $(kubectl -n default get pods -l "run=holiday" -o jsonpath='{.items[0].metadata.name}') -c c1 -- touch /tmp/test
# It works, no problem
```

```bash
# container 2: "-c c2"
$ kubectl exec -ti $(kubectl -n default get pods -l "run=holiday" -o jsonpath='{.items[0].metadata.name}') -c c2 -- touch /tmp/test
# It fails, as expected
touch: /tmp/test: Read-only file system
command terminated with exit code 1
```

