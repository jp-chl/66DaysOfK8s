# Day 33 of #66DaysOfK8s

_Last update: 2021-02-12_

---
Today, I have continued working with network policies, defining rules among pods.

> _Based on: [https://medium.com/better-programming/how-to-secure-kubernetes-using-network-policies-bbb940909364](https://medium.com/better-programming/how-to-secure-kubernetes-using-network-policies-bbb940909364)_

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client
* kubectl client and server v1.19.0

---

## Setup

* K8s cluster already created in GCP from scratch. Check the instructions in this [link](../../week01/day5/README.md).
* Set an alias for kubectl (```alias k=kubectl```).
* This lab does not work on minikube.
* All tests are done in default namespace.

---

## Tasks

* Create 3 Pods, each with its own label (api, bff, backend; respectively).
* Set a network policy for a specific Pod.
* Test communication among Pods.

---

### Create 3 Pods, each with its own label

In the [previous part](../day32/README.md), Network policies were applied to namespaces. In this part, to Pods.

Based on the same image (simple nginx with curl command installed), create 3 Pods each with a unique label.

```bash
$ student@master: k run api --image=ewoutp/docker-nginx-curl -l=app=api
pod/api created
```

```bash
$ student@master: k run bff --image=ewoutp/docker-nginx-curl -l=app=backend
pod/bff created
```

```bash
$ student@master: k run backend --image=ewoutp/docker-nginx-curl -l=app=bff
pod/backend created
```

---

### Set a network policy for a specific Pod

Create a network policy for ```backend``` Pod, only allowing ingress traffic from ```bff``` Pod.

```yaml
# pod-np-ingress-bff-backend.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
#  namespace: 
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: bff
    ports:
    - protocol: TCP
      port: 80
```

```bash
$ student@master: k apply -f yaml/pod-np-ingress-bff-backend.yaml
networkpolicy.networking.k8s.io/backend-policy created
```

```bash
$ student@master: k get networkpolicies
NAME             POD-SELECTOR   AGE
backend-policy   app=backend    33s
```

```bash
$ student@master: k describe networkpolicy backend-policy
```

```yaml
Name:         backend-policy
Namespace:    default
Created on:   2021-02-13 00:11:31 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=backend
  Allowing ingress traffic:
    To Port: 80/TCP
    From:
      PodSelector: app=bff
  Not affecting egress traffic
  Policy Types: Ingress
```

### Test communication among Pods

```bash
$ student@master: k get pods -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
api       1/1     Running   0          10m   192.168.171.118   worker   <none>           <none>
backend   1/1     Running   0          15m   192.168.171.117   worker   <none>           <none>
bff       1/1     Running   0          16m   192.168.171.116   worker   <none>           <none>
```

```bash
export BACKEND_POD_IP=$(k get pod -l=app=backend -o jsonpath='{.items[0].status.podIP}')
export BFF_POD_IP=$(k get pod -l=app=bff -o jsonpath='{.items[0].status.podIP}')
```

```bash
echo $BACKEND_POD_IP
192.168.171.117
```

```bash
echo $BFF_POD_IP
192.168.171.116
```

```api``` Pod to ```backend``` Pod communication **is not** allowed

```bash
$ student@master: k exec -ti api -- curl $BACKEND_POD_IP --max-time 5 | head -4
curl: (28) Connection timed out after 5001 milliseconds
command terminated with exit code 28
```

```bff``` Pod to ```backend``` Pod communication **it is** allowed

```bash
$ student@master: k exec -ti bff -- curl $BACKEND_POD_IP --max-time 5 | head -4
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

---

### Cleanup

```bash
k delete pod api
k delete pod bff
k delete pod backend

k delete -f yaml/pod-np-ingress-bff-backend.yaml
```

```bash
pod "api" deleted
pod "bff" deleted
pod "backend" deleted
networkpolicy.networking.k8s.io "backend-policy" deleted
```

---

## References

* [Network policies (official site)](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
