# Day 32 of #66DaysOfK8s

_Last update: 2021-02-11_

---
Today, I have been working with network policies.

In particular, set the network communication rules among namespaces.

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

---

## Tasks

* Create namespaces, each with its own label.
* Set network policies.
* Test communication among namespaces.

---

### Create namespaces, each with its own label

There will be three namespaces for web, middleware and database; respectively.

```yaml
# namespace web
apiVersion: v1
kind: Namespace
metadata:
  name: web
  labels:
    tier: web

```

```yaml
# namespace middleware
apiVersion: v1
kind: Namespace
metadata:
  name: middleware
  labels:
    tier: middleware
```

```yaml
# namespace database
apiVersion: v1
kind: Namespace
metadata:
  name: database
  labels:
    tier: database
```

Apply namespaces

```bash
$ student@master: k apply -f yaml/web-ns.yaml
namespace/web created
$ student@master: k apply -f yaml/middleware-ns.yaml
namespace/middleware created
$ student@master: k apply -f yaml/database-ns.yaml
namespace/database created
```

---

### Set network policies

Two network policies will be set:

* One for middleware namespace, allowing ```ingress``` traffic from namespaces labeled either as ```tier=web``` or ```tier=middleware```. It also allows ```egress``` traffic to namespaces labeled either ```tier=web``` or ```tier=middleware```. Both rules applied to any Pod (```podSelector: {}```)
* Another for database namespace, with similar rules but for namespaces labeled either as ```tier=middleware``` or ```tier=database```.

```yaml
# Network policy for namespace middleware
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: middlware-network-policy
  namespace: middleware
spec:
  podSelector: {} # All pods in the namespace
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tier: web
    ports:
    - protocol: TCP
      port: 80
  - from:
    - namespaceSelector:
        matchLabels:
          tier: middleware
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 80
  - to:
    - namespaceSelector:
        matchLabels:
          tier: middleware
    ports:
    - protocol: TCP
      port: 80
  - to:
    ports:
    - protocol: UDP
      port: 53 # DNS resolution
```

```yaml
# Network policy for namespace database
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-network-policy
  namespace: database
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tier: middleware
    ports:
    - protocol: TCP
      port: 80
  - from:
    - namespaceSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 80
  - to:
    ports:
    - protocol: UDP
      port: 53
```

```bash
$ student@master: k apply -f yaml/middleware-np.yaml
networkpolicy.networking.k8s.io/middlware-network-policy created
$ student@master: k apply -f yaml/database-np.yaml
networkpolicy.networking.k8s.io/database-network-policy created
```

---

### Test communication among namespaces

Three Pods will be created, simple nginx with curl access.

```bash
$ student@master: k -n web create deploy nginx --image=ewoutp/docker-nginx-curl
deployment.apps/nginx created
$ student@master: k -n middleware create deploy nginx --image=ewoutp/docker-nginx-curl
deployment.apps/nginx created
$ student@master: k -n database create deploy nginx --image=ewoutp/docker-nginx-curl
deployment.apps/nginx created
```

Retrieve Pods IPs:

```bash
$ student@master: k get pods -A -o wide|grep nginx
database      nginx-6d6cb79c77-r6rwb                     1/1     Running   0          40s   192.168.171.109   worker   <none>           <none>
middleware    nginx-6d6cb79c77-fr94f                     1/1     Running   0          41s   192.168.171.108   worker   <none>           <none>
web           nginx-6d6cb79c77-h6rl2                     1/1     Running   0          41s   192.168.171.107   worker   <none>           <none>
```

```bash
$ student@master: export WEB_POD_NAME=$(k -n web get pod -l=app=nginx -o jsonpath='{.items[0].metadata.name}')
$ student@master: export WEB_POD_IP=$(k -n web get pod -l=app=nginx -o jsonpath='{.items[0].status.podIP}')

$ student@master: export MIDDLEWARE_POD_NAME=$(k -n middleware get pod -l=app=nginx -o jsonpath='{.items[0].metadata.name}')
$ student@master: export MIDDLEWARE_POD_IP=$(k -n middleware get pod -l=app=nginx -o jsonpath='{.items[0].status.podIP}')

$ student@master: export DATABASE_POD_NAME=$(k -n database get pod -l=app=nginx -o jsonpath='{.items[0].metadata.name}')
$ student@master: export DATABASE_POD_IP=$(k -n database get pod -l=app=nginx -o jsonpath='{.items[0].status.podIP}')
```

---

Test access from Web namespace to Middleware one.

```bash
$ student@master: k -n web exec -ti $WEB_POD_NAME -- curl $MIDDLEWARE_POD_IP --max-time 5
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
# Output omitted
```

It works as expected.

Now, if we try from web to database tier it should not work.

```bash
$ student@master: k -n web exec -ti $WEB_POD_NAME -- curl $DATABASE_POD_IP --max-time 5
curl: (28) Connection timed out after 5001 milliseconds
command terminated with exit code 28
```

Now, let's try from middleware tier to database one.

```bash
$ student@master: k -n middleware exec -ti $MIDDLEWARE_POD_NAME -- curl $DATABASE_POD_IP --max-time 5
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
# Output omitted
```

---

### Cleanup

```bash
$ student@master: k -n web delete deploy nginx
deployment.apps "nginx" deleted
$ student@master: k -n middleware delete deploy nginx
deployment.apps "nginx" deleted
$ student@master: k -n database delete deploy nginx
deployment.apps "nginx" deleted
```

```bash
$ student@master: k delete -f yaml/.
networkpolicy.networking.k8s.io "database-network-policy" deleted
namespace "database" deleted
networkpolicy.networking.k8s.io "middlware-network-policy" deleted
namespace "middleware" deleted
namespace "web" deleted
```

---

## References

* [Network policies (official site)](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
