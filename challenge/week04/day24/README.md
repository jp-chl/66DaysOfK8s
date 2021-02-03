# Day 24 of #66DaysOfK8s

_Last update: 2021-02-03_

---

Today, I have worked with K8s Rest API using and HTTP proxy.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Setup

* All tests run on minikube.

---

## Tasks

* Setup a HTTP proxy
* Call different K8s Rest APIs by using the HTTP proxy.

---

## Setup a HTTP proxy

Instead of calling K8s API with kubectl, a HTTP proxy can be used.

This proxy can be started with kubectl. By default, it starts on port 8001. We will start it in background (notice the process id)

```bash
$ kubectl proxy --api-prefix=/ & echo $! > ./pid.file
[1] 6712
Starting to serve on 127.0.0.1:8001
```
> _(press enter to come back to the prompt)_

---

## Call different K8s Rest APIs by using the HTTP proxy.

```bash
$ curl http://127.0.0.1:8001/api/
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "192.168.64.73:8443"
    }
  ]
}
```

---

```bash
$ curl http://127.0.0.1:8001/api/v1/namespaces/default/pods
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces/default/pods",
    "resourceVersion": "40995"
  },
  "items": []
}
```

---

Let's start a Pod.

```bash
$ kubectl run nginx --image=nginx
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created
```

```bash
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-598b589c46-gsgpw   1/1     Running   0          36s
```

---

Now, let's get the same information but with the HTTP proxy.

```bash
$ curl http://127.0.0.1:8001/api/v1/namespaces/default/pods
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces/default/pods",
    "resourceVersion": "40455"
  },
  "items": [
    {
      "metadata": {
        "name": "nginx-598b589c46-gsgpw",
        "generateName": "nginx-598b589c46-",
        "namespace": "default",
        "selfLink": "/api/v1/namespaces/default/pods/nginx-598b589c46-gsgpw",
        "uid": "4d536ce3-3f88-4ff0-9769-e3fa5f4be870",
        "resourceVersion": "40400",
        "creationTimestamp": "2021-02-03T22:52:33Z",
        "labels": {
          "pod-template-hash": "598b589c46",
          "run": "nginx"
        },
# Output omitted
```

---

## Cleanup

Stop proxy:

```bash
$ kill -9 $(cat ./pid.file)
[1]  + 6712 killed     kubectl proxy --api-prefix=/
```

Delete pod:

```bash
$ kubectl delete deployment nginx
deployment.apps "nginx" deleted
```

---

## References

* [Use an HTTP Proxy to Access the Kubernetes API(official site)](https://kubernetes.io/docs/tasks/extend-kubernetes/http-proxy-access-api/)

* [Proxies in Kubernetes (official site)](https://kubernetes.io/docs/concepts/cluster-administration/proxies/)
