# Day 51 of #66DaysOfK8s

_Last update: 2021-03-03_

---
Today, I have worked with liveness probe.

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

* Configure a liveness probe for a Pod
* Set a not-healthy status on Pod
* Test Pod restarting

---

## Configure a liveness probe for a Pod

_"The kubelet uses liveness probes to know when to restart a container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs."_ -- (official site)

In order to automatically check Pod's health, you can set a Pod's liveness probe in order to certify whether Pod is alive.

In the Pod's container spec you can add liveness block to do it. One of the most common ways to handle this is to define a container endpoint that the Kubelet pings periodically. It will be considered healthy while it responds a 200 http code, or unhealthy otherwise.

A typical liveness probe looks like:

```yaml
apiVersion: v1
kind: Pod
# Output omitted
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
# Output omitted
    livenessProbe:
      httpGet:
        path: /health-endpoint # Container's endpoint to check liveness
        port: 8080
      initialDelaySeconds: 3 # Initial delay before periodically check liveness
      periodSeconds: 3 # Periodicity check
```

Where the endpoint is defined in ```path``` and ```port``` variables. Besides, a startup delay in seconds can be set by changing ```initialDelaySeconds```. K8s will test liveness every ```periodSeconds```.

---

Let's start a utility Pod available at [docker hub](https://hub.docker.com/r/nectiadocker2000/podtesterspring), that defines an endpoint to check health, and [other ones](https://github.com/jp-chl/podtesterspring) to emulate misbehaving. 

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: tester
  name: tester
  namespace: default
spec:
  containers:
  - image: nectiadocker2000/podtesterspring:v2
    name: tester
    livenessProbe:
      httpGet:
        path: /healthz/ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

```bash
$ k apply -f pod.yaml
pod/tester created
```

Test liveness endpoint:

```bash
$ k exec -ti tester -- curl -i localhost:8080/healthz/ready
HTTP/1.1 200
Content-Type: text/plain;charset=UTF-8
Content-Length: 10
Date: Thu, 04 Mar 2021 02:47:04 GMT

I am alive
```

---

## Set a not-healthy status on Pod

If we shoot at /misbehave endpoint, liveness one will return a non 200 http code.

```bash
$ k exec -ti tester -- curl -i localhost:8080/misbehave
HTTP/1.1 200
Content-Type: text/plain;charset=UTF-8
Content-Length: 14
Date: Thu, 04 Mar 2021 02:48:40 GMT

Misbehaving...
```

```bash
$ k exec -ti tester -- curl -i localhost:8080/healthz/ready
HTTP/1.1 503
Content-Type: text/plain;charset=UTF-8
Content-Length: 9
Date: Thu, 04 Mar 2021 02:48:45 GMT
Connection: close

I am down
```

```bash
$ k delete -f pod.yaml
deployment.apps "hog" deleted
```

---

## Test Pod restarting

After three attempts the Pod will be restarted is non 200 http code is returned from liveness configured endpoint.

```bash
$ k apply -f pod.yaml
pod/tester created
```

```bash
$ k get pods --watch
NAME     READY   STATUS              RESTARTS   AGE
tester   0/1     ContainerCreating   0          0s
tester   1/1     Running             0          2s
```

```bash
k exec -ti tester -- curl -s -o /dev/null -i -w "%{http_code}\n" localhost:8080/healthz/ready
200
```

```bash
$ k exec -ti tester -- curl -s -o /dev/null -i -w "%{http_code}\n" localhost:8080/misbehave
200
```

```bash
$ k exec -ti tester -- curl -s -o /dev/null -i -w "%{http_code}\n" localhost:8080/healthz/ready
503
```

```bash
$ k describe pod tester
# Output omitted
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  53s                default-scheduler  Successfully assigned default/tester to minikube
  Warning  Unhealthy  43s (x2 over 48s)  kubelet, minikube  Liveness probe failed: Get "http://172.17.0.3:8080/healthz/ready": dial tcp 172.17.0.3:8080: connect: connection refused
  Warning  Unhealthy  3s (x3 over 13s)   kubelet, minikube  Liveness probe failed: HTTP probe failed with statuscode: 503
  Normal   Killing    3s                 kubelet, minikube  Container tester failed liveness probe, will be restarted
  Normal   Pulled     2s (x2 over 52s)   kubelet, minikube  Container image "nectiadocker2000/podtesterspring:v2" already present on machine
  Normal   Created    2s (x2 over 52s)   kubelet, minikube  Created container tester
  Normal   Started    2s (x2 over 52s)   kubelet, minikube  Started container tester
```

```bash
$ k get pods --watch
NAME     READY   STATUS              RESTARTS   AGE
tester   0/1     ContainerCreating   0          0s
tester   1/1     Running             0          2s
tester   1/1     Running             1          51s
```

```bash
$ k exec -ti tester -- curl -s -o /dev/null -i -w "%{http_code}\n" localhost:8080/misbehave
200
```

```bash
$ k describe pod tester
# Output omitted
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  2m7s                default-scheduler  Successfully assigned default/tester to minikube
  Warning  Unhealthy  67s (x4 over 2m2s)  kubelet, minikube  Liveness probe failed: Get "http://172.17.0.3:8080/healthz/ready": dial tcp 172.17.0.3:8080: connect: connection refused
  Warning  Unhealthy  2s (x6 over 87s)    kubelet, minikube  Liveness probe failed: HTTP probe failed with statuscode: 503
  Normal   Killing    2s (x2 over 77s)    kubelet, minikube  Container tester failed liveness probe, will be restarted
  Normal   Pulled     1s (x3 over 2m6s)   kubelet, minikube  Container image "nectiadocker2000/podtesterspring:v2" already present on machine
  Normal   Created    1s (x3 over 2m6s)   kubelet, minikube  Created container tester
  Normal   Started    1s (x3 over 2m6s)   kubelet, minikube  Started container tester
```

```bash
$ k get pods --watch
NAME     READY   STATUS              RESTARTS   AGE
tester   0/1     ContainerCreating   0          0s
tester   1/1     Running             0          2s
tester   1/1     Running             1          51s
tester   1/1     Running             2          2m6s
```

---

## Cleanup

```bash
$ k delete -f pod.yaml
pod "tester" deleted
```

---

## References

* [Configure Liveness, Readiness and Startup Probes (official site)](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
