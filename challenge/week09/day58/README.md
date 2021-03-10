# Day 58 of #66DaysOfK8s

_Last update: 2021-03-09_

---
Today I have worked in the 3rd part of Deployment management (no downtime example).

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
* Expose the deployment as a service, hit the endpoint continously while changing its endpoint
* Ensure no downtime

---

## Create a deployment with several replicas

Refer to [part 1](../../week08/day56/) for all the details.

Initially, we'll be using quay.io/rhdevelopers/quarkus-demo:v1 image, with the default strategy and 6 replicas.

```yaml
# deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myapp
  name: myapp
  namespace: default
spec:
  replicas: 6
  selector:
    matchLabels:
      app: myapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - image: quay.io/rhdevelopers/quarkus-demo:v1
        imagePullPolicy: IfNotPresent
        name: demo
```

```bash
$ k apply -f yaml/deploy.yaml
deployment.apps/myapp created
```

---

## Expose the deployment as a service, hit the endpoint continously while changing its endpoint

```bash
$ k expose deployment myapp --port=8080 --type=LoadBalancer
service/myapp exposed
```

Test service access. Keep while curl cycle running.

```bash
$ while true; do curl "http://$(minikube ip):$(k get svc myapp -o jsonpath='{.spec.ports[0].nodePort}')"; sleep .5; done
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-kml4r:1
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-cq28s:1
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-j4qmw:1
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-5hl79:1
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-5hl79:2
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-sdcxm:1
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-kml4r:2
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-j4qmw:2
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-5hl79:3
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-j4f2q:1
Supersonic Subatomic Java with Quarkus myapp-784dbf86d9-j4f2q:2
...
```

Now, roll out a new image (quay.io/rhdevelopers/myboot:v1) and notice some downtime.

```bash
# Replace image from quarkus-demo:v1 to myboot:v1
$ vi yaml/deploy.yaml

# ...
        image: quay.io/rhdevelopers/myboot:v1
# ...
```

```bash
$ k apply -f yaml/deploy.yaml
deployment.apps/myapp created
```

```bash
$ while true; do curl "http://$(minikube ip):$(k get svc myapp -o jsonpath='{.spec.ports[0].nodePort}')"; sleep .5; done
Supersonic Subatomic Java with Quarkus myapp-5877cdf5bb-8lxfn:46
Supersonic Subatomic Java with Quarkus myapp-5877cdf5bb-8lxfn:47
Supersonic Subatomic Java with Quarkus myapp-5877cdf5bb-698r6:45
Supersonic Subatomic Java with Quarkus myapp-5877cdf5bb-698r6:46
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
Supersonic Subatomic Java with Quarkus myapp-5877cdf5bb-698r6:47
Supersonic Subatomic Java with Quarkus myapp-5877cdf5bb-698r6:48
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
Supersonic Subatomic Java with Quarkus myapp-5877cdf5bb-698r6:49
Supersonic Subatomic Java with Quarkus myapp-5877cdf5bb-698r6:50
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused

...

curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
Aloha from Spring Boot! 1 on myapp-74b76d584b-cgws2
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
Aloha from Spring Boot! 2 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 3 on myapp-74b76d584b-cgws2
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
curl: (7) Failed to connect to 192.168.64.118 port 30945: Connection refused
Aloha from Spring Boot! 1 on myapp-74b76d584b-7t6kf
Aloha from Spring Boot! 1 on myapp-74b76d584b-8j2z5
Aloha from Spring Boot! 2 on myapp-74b76d584b-8j2z5
Aloha from Spring Boot! 2 on myapp-74b76d584b-7t6kf
Aloha from Spring Boot! 3 on myapp-74b76d584b-8j2z5
Aloha from Spring Boot! 3 on myapp-74b76d584b-7t6kf
...
```

---

## Ensure no downtime

Add liveness and readiness to ensure no downtime. Set the appropriate timing, especially for readiness in a Spring Boot container (which normally does not start fast enough).

Set liveness and readiness to "```/```" endpoint.

```bash
# Replace image from myboot:v1 to myboot:v2, and add liveness and readiness probes
$ vi yaml/deploy.yaml

# ...
        image: quay.io/rhdevelopers/myboot:v2
# ...
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 45
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 45
          periodSeconds: 10

```

```bash
$ k apply -f yaml/deploy.yaml
deployment.apps/myapp created
```

This time you won't notice a downtime whatsoever.

```bash
$ while true; do curl "http://$(minikube ip):$(k get svc myapp -o jsonpath='{.spec.ports[0].nodePort}')"; sleep .5; done
...
Aloha from Spring Boot! 26 on myapp-74b76d584b-6ccj7
Aloha from Spring Boot! 27 on myapp-74b76d584b-6ccj7
Aloha from Spring Boot! 33 on myapp-74b76d584b-4knn6
Aloha from Spring Boot! 34 on myapp-74b76d584b-4knn6
Aloha from Spring Boot! 28 on myapp-74b76d584b-6ccj7
Aloha from Spring Boot! 27 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 29 on myapp-74b76d584b-7t6kf
Aloha from Spring Boot! 28 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 31 on myapp-74b76d584b-8j2z5
Aloha from Spring Boot! 29 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 3 on myapp-7ff5664fff-r9g9w
Aloha from Spring Boot! 32 on myapp-74b76d584b-8j2z5
Aloha from Spring Boot! 35 on myapp-74b76d584b-4knn6
Aloha from Spring Boot! 30 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 30 on myapp-74b76d584b-7t6kf
Bonjour from Spring Boot! 3 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 2 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 31 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 32 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 31 on myapp-74b76d584b-7t6kf
Bonjour from Spring Boot! 4 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 4 on myapp-7ff5664fff-pg4lc
Bonjour from Spring Boot! 5 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 33 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 34 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 5 on myapp-7ff5664fff-mdrpd
Aloha from Spring Boot! 32 on myapp-74b76d584b-7t6kf
Bonjour from Spring Boot! 7 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 33 on myapp-74b76d584b-7t6kf
Aloha from Spring Boot! 34 on myapp-74b76d584b-7t6kf
Bonjour from Spring Boot! 6 on myapp-7ff5664fff-r9g9w
Bonjour from Spring Boot! 9 on myapp-7ff5664fff-pg4lc
Bonjour from Spring Boot! 7 on myapp-7ff5664fff-r9g9w
Bonjour from Spring Boot! 8 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 9 on myapp-7ff5664fff-r9g9w
Bonjour from Spring Boot! 11 on myapp-7ff5664fff-r9g9w
Aloha from Spring Boot! 35 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 35 on myapp-74b76d584b-7t6kf
Bonjour from Spring Boot! 12 on myapp-7ff5664fff-r9g9w
Bonjour from Spring Boot! 11 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 13 on myapp-7ff5664fff-r9g9w
Aloha from Spring Boot! 36 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 14 on myapp-7ff5664fff-r9g9w
Bonjour from Spring Boot! 12 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 15 on myapp-7ff5664fff-r9g9w
Bonjour from Spring Boot! 16 on myapp-7ff5664fff-r9g9w
Aloha from Spring Boot! 37 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 13 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 12 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 38 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 14 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 39 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 15 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 40 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 19 on myapp-7ff5664fff-r9g9w
Aloha from Spring Boot! 41 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 42 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 43 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 36 on myapp-74b76d584b-7t6kf
Bonjour from Spring Boot! 16 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 17 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 44 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 18 on myapp-7ff5664fff-pg4lc
Bonjour from Spring Boot! 17 on myapp-7ff5664fff-mdrpd
Aloha from Spring Boot! 45 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 20 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 21 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 20 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 46 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 22 on myapp-7ff5664fff-mdrpd
Aloha from Spring Boot! 37 on myapp-74b76d584b-7t6kf
Bonjour from Spring Boot! 22 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 38 on myapp-74b76d584b-7t6kf
Aloha from Spring Boot! 39 on myapp-74b76d584b-7t6kf
Aloha from Spring Boot! 47 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 24 on myapp-7ff5664fff-r9g9w
Bonjour from Spring Boot! 23 on myapp-7ff5664fff-pg4lc
Aloha from Spring Boot! 48 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 49 on myapp-74b76d584b-cgws2
Aloha from Spring Boot! 50 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 25 on myapp-7ff5664fff-mdrpd
Bonjour from Spring Boot! 3 on myapp-7ff5664fff-htmll
Bonjour from Spring Boot! 25 on myapp-7ff5664fff-pg4lc
Bonjour from Spring Boot! 26 on myapp-7ff5664fff-pg4lc
Bonjour from Spring Boot! 26 on myapp-7ff5664fff-mdrpd
Aloha from Spring Boot! 51 on myapp-74b76d584b-cgws2
Bonjour from Spring Boot! 25 on myapp-7ff5664fff-r9g9w
Bonjour from Spring Boot! 3 on myapp-7ff5664fff-5njqm
Bonjour from Spring Boot! 5 on myapp-7ff5664fff-htmll
Bonjour from Spring Boot! 28 on myapp-7ff5664fff-r9g9w
Bonjour from Spring Boot! 4 on myapp-7ff5664fff-5njqm
Bonjour from Spring Boot! 5 on myapp-7ff5664fff-5njqm
Bonjour from Spring Boot! 28 on myapp-7ff5664fff-pg4lc
Bonjour from Spring Boot! 29 on myapp-7ff5664fff-r9g9w
...
```

---

## Cleanup

```bash
$ k delete all --all
pod "myapp-7ff5664fff-5njqm" deleted
pod "myapp-7ff5664fff-htmll" deleted
pod "myapp-7ff5664fff-mdrpd" deleted
pod "myapp-7ff5664fff-pg4lc" deleted
pod "myapp-7ff5664fff-r9g9w" deleted
pod "myapp-7ff5664fff-vtfxb" deleted
service "kubernetes" deleted
service "myapp" deleted
deployment.apps "myapp" deleted
replicaset.apps "myapp-5877cdf5bb" deleted
replicaset.apps "myapp-74b76d584b" deleted
replicaset.apps "myapp-7ff5664fff" deleted
```

---

## References

* [Deployment (official site)](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
