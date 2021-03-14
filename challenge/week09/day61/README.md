# Day 61 of #66DaysOfK8s

_Last update: 2021-03-12_

---
Today I have worked with an example of a Cassandra cluster with Stateful Sets.

_Based on an [Github article by Jan Šafránek](https://github.com/jsafrane/caas/)_.

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

* Create a cassandra cluster
* Test cassandra access and a service accessing it

---

## Create a cassandra cluster

The following examples will deploy a Headless Service serving 3 replicas of a cassandra stateful db.

```yaml
# cassandra.yaml
# Headless Service that creates DNS entries for Cassnadra pods, i.e.
# for pods with label "app=cassandra".
apiVersion: v1
kind: Service
metadata:
  labels:
    app: cassandra
  name: cassandra
spec:
  clusterIP: None
  ports:
  - port: 9042
  selector:
    app: cassandra
---

# Cassandra StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cassandra
  labels:
    app: cassandra
spec:
  serviceName: cassandra
  replicas: 3
  selector:
    matchLabels:
      app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      # Do not kill Cassandra pods immediately, give them some time to finish gracefully.
      terminationGracePeriodSeconds: 1800

      containers:
      - name: cassandra
        image: gcr.io/google-samples/cassandra:v13
        imagePullPolicy: Always
        ports:
        - containerPort: 7000
          name: intra-node
        - containerPort: 7001
          name: tls-intra-node
        - containerPort: 7199
          name: jmx
        - containerPort: 9042
          name: cql
        resources:
          # Minimum requirement.
          requests:
            cpu: "100m" # 0.1 CPU
            memory: 1Gi
          # Maximum req.
          limits:
            cpu: "4"    # 4 CPUs
            memory: 1Gi
        # Cassandra needs capability to lock shared mem.
        securityContext:
          capabilities:
            add:
              - IPC_LOCK
        env:
          - name: MAX_HEAP_SIZE
            value: 512M
          - name: HEAP_NEWSIZE
            value: 100M
          - name: CASSANDRA_SEEDS
            value: "cassandra-0.cassandra.default.svc.cluster.local"
          - name: CASSANDRA_CLUSTER_NAME
            value: "K8Demo"
          - name: CASSANDRA_DC
            value: "DC1-K8Demo"
          - name: CASSANDRA_RACK
            value: "Rack1-K8Demo"
          - name: CASSANDRA_RING_DELAY
            value: "5000"
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
        # Before killing container, execute this command so the daemon can exit gracefully.
        lifecycle:
          preStop:
            exec:
              command: 
              - /bin/sh
              - -c
              - nodetool drain
        # Wait until this command succeeds before declaring this pod read to serve requests.
        readinessProbe:
          exec:
            command:
            - /bin/bash
            - -c
            - /ready-probe.sh
          initialDelaySeconds: 15
          timeoutSeconds: 5
        # These volume mounts are persistent. They are like inline claims,
        # but not exactly because the names need to match exactly one of
        # the stateful pod volumes.
        volumeMounts:
        - name: cassandra-data
          mountPath: /cassandra_data
  # These are converted to volume claims by the controller
  # and mounted at the paths mentioned above.
  volumeClaimTemplates:
  - metadata:
      name: cassandra-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

```bash
$ kubectl apply -f yaml/cassandra.yaml
service/cassandra created
statefulset.apps/cassandra created
```

```bash
$ kubectl get svc,pod,pv,pvc,statefulset

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/cassandra    ClusterIP   None         <none>        9042/TCP   8m10s
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP    14m

NAME              READY   STATUS    RESTARTS   AGE
pod/cassandra-0   1/1     Running   0          8m10s
pod/cassandra-1   1/1     Running   0          7m32s
pod/cassandra-2   1/1     Running   0          7m8s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS
CLAIM                                STORAGECLASS   REASON   AGE
persistentvolume/pvc-7b622b3b-cdcb-49aa-9a48-1b6260dc45a1   1Gi        RWO            Delete           Bound
default/cassandra-data-cassandra-0   standard                8m10s
persistentvolume/pvc-89c3ed96-5d7d-488e-8fe2-a00f845777da   1Gi        RWO            Delete           Bound
default/cassandra-data-cassandra-1   standard                7m32s
persistentvolume/pvc-faf8cc8e-fcb3-4f71-a66f-1a4e142c8be0   1Gi        RWO            Delete           Bound
default/cassandra-data-cassandra-2   standard                7m8s

NAME                                               STATUS   VOLUME                                     CAPACITY
  ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/cassandra-data-cassandra-0   Bound    pvc-7b622b3b-cdcb-49aa-9a48-1b6260dc45a1   1Gi
  RWO            standard       8m10s
persistentvolumeclaim/cassandra-data-cassandra-1   Bound    pvc-89c3ed96-5d7d-488e-8fe2-a00f845777da   1Gi
  RWO            standard       7m32s
persistentvolumeclaim/cassandra-data-cassandra-2   Bound    pvc-faf8cc8e-fcb3-4f71-a66f-1a4e142c8be0   1Gi
  RWO            standard       7m8s

NAME                         READY   AGE
statefulset.apps/cassandra   3/3     8m10
```

---

## Test cassandra access and a service accessing it

```bash
k get pods -o wide                                     ✔  at minikube ⎈  at 08:32:19 
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
cassandra-0             1/1     Running   0          15m   172.17.0.3   minikube   <none>           <none>
cassandra-1             1/1     Running   0          14m   172.17.0.4   minikube   <none>           <none>
cassandra-2             1/1     Running   0          14m   172.17.0.5   minikube   <none>           <none>
```

```bash
$ k exec cassandra-0 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  76.03 KiB  32           100.0%            f487a2ad-a00f-4323-ad07-5f43c1e3426d  Rack1-K8Demo
UN  172.17.0.5  96.54 KiB  32           100.0%            a2dc3580-212a-47ad-98ea-b4df2f392cce  Rack1-K8Demo
UN  172.17.0.4  87.64 KiB  32           100.0%            8218874a-7f35-4181-b24b-41cb58d5a1b5  Rack1-K8Demo
```

```bash
$ k exec cassandra-1 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  76.03 KiB  32           100.0%            f487a2ad-a00f-4323-ad07-5f43c1e3426d  Rack1-K8Demo
UN  172.17.0.5  96.54 KiB  32           100.0%            a2dc3580-212a-47ad-98ea-b4df2f392cce  Rack1-K8Demo
UN  172.17.0.4  87.64 KiB  32           100.0%            8218874a-7f35-4181-b24b-41cb58d5a1b5  Rack1-K8Demo
```

```bash
$ k exec cassandra-2 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  76.03 KiB  32           100.0%            f487a2ad-a00f-4323-ad07-5f43c1e3426d  Rack1-K8Demo
UN  172.17.0.5  96.54 KiB  32           100.0%            a2dc3580-212a-47ad-98ea-b4df2f392cce  Rack1-K8Demo
UN  172.17.0.4  87.64 KiB  32           100.0%            8218874a-7f35-4181-b24b-41cb58d5a1b5  Rack1-K8Demo
```

---

```yaml
# caas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: caas
spec:
  replicas: 1
  selector:
    matchLabels:
      app: caas
  template:
    metadata:
      labels:
        app: caas
    spec:
      containers:
        - name: caas
          image: quay.io/jsafrane/caas:latest
          ports:
          - containerPort: 80
            name: http
          imagePullPolicy: Always
          env:
            - name: CASSANDRA_ADDRESS
              value: "cassandra.default.svc.cluster.local"
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: caas
  name: caas
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: caas
```

```bash
$ k apply -f yaml/caas.yaml
deployment.apps/caas created
service/caas created
```

---

```bash
$ curl http://$(minikube ip):$(k get svc caas -o jsonpath='{.spec.ports[0].nodePort}')/FirstToken/html
```

```html
<html>
  <head><title>FirstToken</title></head>
<body>
<h1>Counter: FirstToken, value: 1</h1>
<pre>
Web server: caas-54977c69d7-kjv2s
Queries:

  DB server: 172.17.0.5
  Query:     UPDATE counter SET value=value&#43;1 WHERE name = ?
  Attempts:  1
  Time:      114.659985 miliseconds

  DB server: 172.17.0.3
  Query:     SELECT name, value FROM counter WHERE name=? LIMIT 1
  Attempts:  1
  Time:      51.686838 miliseconds

</pre>
</body>
</html>
```

---

```bash
$ curl http://$(minikube ip):$(k get svc caas -o jsonpath='{.spec.ports[0].nodePort}')/AnotherToken/html
```

```html
<html>
  <head><title>AnotherToken</title></head>
<body>
<h1>Counter: AnotherToken, value: 1</h1>
<pre>
Web server: caas-54977c69d7-kjv2s
Queries:

  DB server: 172.17.0.3
  Query:     UPDATE counter SET value=value&#43;1 WHERE name = ?
  Attempts:  1
  Time:      18.329775 miliseconds

  DB server: 172.17.0.4
  Query:     SELECT name, value FROM counter WHERE name=? LIMIT 1
  Attempts:  1
  Time:      26.480843 miliseconds

</pre>
</body>
</html>
```

---

```bash
$ curl http://$(minikube ip):$(k get svc caas -o jsonpath='{.spec.ports[0].nodePort}')/FirstToken/html
```

```html

<html>
  <head><title>FirstToken</title></head>
<body>
<h1>Counter: FirstToken, value: 3</h1>
<pre>
Web server: caas-54977c69d7-kjv2s
Queries:

  DB server: 172.17.0.5
  Query:     UPDATE counter SET value=value&#43;1 WHERE name = ?
  Attempts:  1
  Time:      10.186302 miliseconds

  DB server: 172.17.0.3
  Query:     SELECT name, value FROM counter WHERE name=? LIMIT 1
  Attempts:  1
  Time:      5.808416 miliseconds

</pre>
</body>
</html>
```

---

Test graceful failure, scaling and query the db.

```bash
k delete pod cassandra-1; k get pods -w
pod "cassandra-1" deleted

NAME                    READY   STATUS              RESTARTS   AGE
caas-54977c69d7-kjv2s   1/1     Running             0          7m23s
cassandra-0             1/1     Running             0          21m
cassandra-1             0/1     ContainerCreating   0          0s
cassandra-2             1/1     Running             0          20m
cassandra-1             0/1     Running             0          3s
cassandra-1             1/1     Running             0          25s
```

---

```bash
$ k scale statefulset cassandra --replicas=4
statefulset.apps/cassandra scaled
```

```bash
$ k exec cassandra-0 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  76.03 KiB  32           100.0%            f487a2ad-a00f-4323-ad07-5f43c1e3426d  Rack1-K8Demo
UN  172.17.0.5  96.54 KiB  32           100.0%            a2dc3580-212a-47ad-98ea-b4df2f392cce  Rack1-K8Demo
UN  172.17.0.4  131.34 KiB  32           100.0%            8218874a-7f35-4181-b24b-41cb58d5a1b5  Rack1-K8Demo
```

```bash
$ k get pods cassandra-3
NAME          READY   STATUS    RESTARTS   AGE
cassandra-3   0/1     Running   0          26s
```

```bash
$ k exec cassandra-0 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  80.94 KiB  32           90.3%             f487a2ad-a00f-4323-ad07-5f43c1e3426d  Rack1-K8Demo
UN  172.17.0.5  96.54 KiB  32           77.3%             a2dc3580-212a-47ad-98ea-b4df2f392cce  Rack1-K8Demo
UN  172.17.0.4  131.34 KiB  32           69.1%             8218874a-7f35-4181-b24b-41cb58d5a1b5  Rack1-K8Demo
UN  172.17.0.7  36.02 KiB  32           63.2%             d4ef1629-2f20-4ad7-a560-7fbafeefd793  Rack1-K8Demo
```

---

```bash
$ k scale statefulset cassandra --replicas=3
statefulset.apps/cassandra scaled
```

```bash
$ k exec cassandra-0 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  80.94 KiB  32           90.3%             f487a2ad-a00f-4323-ad07-5f43c1e3426d  Rack1-K8Demo
UN  172.17.0.5  101.45 KiB  32           77.3%             a2dc3580-212a-47ad-98ea-b4df2f392cce  Rack1-K8Demo
UN  172.17.0.4  131.34 KiB  32           69.1%             8218874a-7f35-4181-b24b-41cb58d5a1b5  Rack1-K8Demo
DN  172.17.0.7  109.19 KiB  32           63.2%             d4ef1629-2f20-4ad7-a560-7fbafeefd793  Rack1-K8Demo
```

```bash
$ k get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
caas-54977c69d7-kjv2s   1/1     Running   0          14m     172.17.0.6   minikube   <none>           <none>
cassandra-0             1/1     Running   0          28m     172.17.0.3   minikube   <none>           <none>
cassandra-1             1/1     Running   0          6m42s   172.17.0.4   minikube   <none>           <none>
cassandra-2             1/1     Running   0          27m     172.17.0.5   minikube   <none>           <none>
```

A node has to be removed manually. In this example it is the one linked to the IP ```172.17.0.7```.

```bash
$ k exec cassandra-0 -- nodetool removenode d4ef1629-2f20-4ad7-a560-7fbafeefd793
```

```bash
$ k exec cassandra-0 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  87.64 KiB  32           100.0%            f487a2ad-a00f-4323-ad07-5f43c1e3426d  Rack1-K8Demo
UN  172.17.0.5  101.45 KiB  32           100.0%            a2dc3580-212a-47ad-98ea-b4df2f392cce  Rack1-K8Demo
UN  172.17.0.4  131.34 KiB  32           100.0%            8218874a-7f35-4181-b24b-41cb58d5a1b5  Rack1-K8Demo
```

---

```bash
$ k run --restart=Never --rm -ti --generator=run-pod/v1 cqlsh --image=cassandra:latest -- cqlsh cassandra-0.cassandra.default.svc.cluster.local -k caas
If you don't see a command prompt, try pressing enter.
Connected to K8Demo at cassandra-0.cassandra.default.svc.cluster.local:9042.
[cqlsh 5.0.1 | Cassandra 3.11.2 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
```

```sql
cqlsh:caas>
cqlsh:caas> select * from counter;

 name         | value
--------------+-------
   FirstToken |     3
 AnotherToken |     1

(2 rows)
cqlsh:caas> exit
pod "cqlsh" deleted
```

---

## Cleanup

```bash
$ k delete -f yaml/.
deployment.apps "caas" deleted
service "caas" deleted
service "cassandra" deleted
statefulset.apps "cassandra" deleted
```

```bash
$ k delete pvc cassandra-data-cassandra-0; k delete pvc cassandra-data-cassandra-1; k delete pvc cassandra-data-cassandra-2; k delete pvc cassandra-data-cassandra-3
persistentvolumeclaim "cassandra-data-cassandra-0" deleted
persistentvolumeclaim "cassandra-data-cassandra-1" deleted
persistentvolumeclaim "cassandra-data-cassandra-2" deleted
persistentvolumeclaim "cassandra-data-cassandra-3" deleted
```

---

## References

* [StatefulSets (official site)](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
