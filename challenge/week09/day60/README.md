# Day 60 of #66DaysOfK8s

_Last update: 2021-03-11_

---
Today I have worked with Stateful Sets.

_Based on an [Github article by Bob Killen](https://github.com/mrbobbytables/k8s-intro-tutorials/blob/master/workloads/README.md)_.

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

* Create a simple StatefulSet
* Test service discovery with a Headless service

---

## Create a simple stateful set

Like the [first part](../day59), create a StatefulSet with 3 replicas.

```yaml
# StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sts-example
spec:
  replicas: 3
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: stateful
      v: v1
  serviceName: app
  updateStrategy:
    type: OnDelete
  template:
    metadata:
      labels:
        app: stateful
        v: v1
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: standard
      resources:
        requests:
          storage: 1Gi
```

The last yaml spawns 3 Pods, 3 PVC and 3 PV.

```bash
$ kubectl apply -f yaml/ss.yaml
statefulset.apps/sts-example created
```

---

## Test service discovery with a Headless service

After successful StatefulSet creation, spawn a Headless service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  clusterIP: None # Headless service indicator
  selector:
    app: stateful
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```bash
$ kubectl apply -f yaml/headless.yaml
service/app created
```

A ```nslookup``` to a Headless service points to an ```A record``` for each instance of the StatefulSet.

```bash
$ k get pods -o wide
NAME            READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
sts-example-0   1/1     Running   0          4m20s   172.17.0.3   minikube   <none>           <none>
sts-example-1   1/1     Running   0          4m14s   172.17.0.4   minikube   <none>           <none>
sts-example-2   1/1     Running   0          4m9s    172.17.0.5   minikube   <none>           <none>
```

```bash
$ k exec sts-example-0 -- nslookup app.default.svc.cluster.local
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	app.default.svc.cluster.local
Address: 172.17.0.4
Name:	app.default.svc.cluster.local
Address: 172.17.0.5
Name:	app.default.svc.cluster.local
Address: 172.17.0.3
```

Similarly, one instance can be queried directly.

```bash
$ k exec sts-example-0 -- nslookup sts-example-1.app.default.svc.cluster.local
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	sts-example-1.app.default.svc.cluster.local
Address: 172.17.0.4
```

---

## Cleanup

```bash
$ k delete all --all
pod "sts-example-0" deleted
pod "sts-example-1" deleted
pod "sts-example-2" deleted
service "app" deleted
service "kubernetes" deleted
statefulset.apps "sts-example" deleted
```

```bash
$ k delete pvc www-sts-example-0; k delete pvc www-sts-example-1; k delete pvc www-sts-example-2
persistentvolumeclaim "www-sts-example-0" deleted
persistentvolumeclaim "www-sts-example-1" deleted
persistentvolumeclaim "www-sts-example-2" deleted
```

---

## References

* [StatefulSets (official site)](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
