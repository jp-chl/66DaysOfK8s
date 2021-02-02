# Day 22 of #66DaysOfK8s

_Last update: 2021-02-01_

---

Today, I have worked in understanding API calls.


#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0
* jq: 1.6
* dtruss (linux strace equivalent)

---

## Setup

* A minikube local cluster has to be already configured and running.
* All tests run on minikube.

---

## Tasks

* Identify cache directory for connected cluster.
* Understand some APIs.

---

### Identify cache directory for connected cluster

Get Kubernetes Pods, but catch syscall with ```dtruss``` command. Its output is very long. Find some lines that begin with "```open(```" pointing out to \<home dir\>/.kube/cache/discovery/\<server\>/ directory.

```bash
$ sudo dtruss kubectl get pods
dtrace: system integrity protection is on, some features will not be available

SYSCALL(args)            = return

# Output omitted

open("/Users/xxx/.kube/cache/discovery/192.168.64.72_8443/events.k8s.io/v1/serverresources.json\0", 0x1000000, 0x0)

# Output omitted
```

---

### Understand some APIs

Now, explore serverresources.json file located in ```\<home dir\>/.kube/cache/discovery/\<server\>/events.k8s.io/v1``` directory.

```bash
$ cat /Users/xxx/.kube/cache/discovery/192.168.64.72_8443/events.k8s.io/v1/serverresources.json | jq
```

```json
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "events.k8s.io/v1",
  "resources": [
    {
      "name": "events",
      "singularName": "",
      "namespaced": true,
      "kind": "Event",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "ev"
      ],
      "storageVersionHash": "r2yiGXH7wu8="
    }
  ]
}
```

You can see the event is the API kind, it has some verbs as options and a short name is "_ev_".

---

If you explore that directory you will find more APIs.

```bash
$ cd /Users/xxx/.kube/cache/discovery/192.168.64.72_8443
```

```bash
$ tree -L 2
├── admissionregistration.k8s.io
│   ├── v1
│   └── v1beta1
├── apiextensions.k8s.io
│   ├── v1
│   └── v1beta1
├── apiregistration.k8s.io
│   ├── v1
│   └── v1beta1
├── apps
│   └── v1
├── authentication.k8s.io
│   ├── v1
│   └── v1beta1
├── authorization.k8s.io
│   ├── v1
│   └── v1beta1
├── autoscaling
│   ├── v1
│   ├── v2beta1
│   └── v2beta2
├── batch
│   ├── v1
│   └── v1beta1
├── certificates.k8s.io
│   ├── v1
│   └── v1beta1
├── coordination.k8s.io
│   ├── v1
│   └── v1beta1
├── discovery.k8s.io
│   └── v1beta1
├── events.k8s.io
│   ├── v1
│   └── v1beta1
├── extensions
│   └── v1beta1
├── networking.k8s.io
│   ├── v1
│   └── v1beta1
├── node.k8s.io
│   └── v1beta1
├── policy
│   └── v1beta1
├── rbac.authorization.k8s.io
│   ├── v1
│   └── v1beta1
├── scheduling.k8s.io
│   ├── v1
│   └── v1beta1
├── servergroups.json
├── storage.k8s.io
│   ├── v1
│   └── v1beta1
└── v1
    └── serverresources.json
```

For example ```Pod``` in ```v1 API``` is described as follows.

```bash
$ cat v1/serverresources.json|jq|grep "pods"
```

```bash
# Output omitted
```

```json
{
  "name": "pods",
  "singularName": "",
  "namespaced": true,
  "kind": "Pod",
  "verbs": [
    "create",
    "delete",
    "deletecollection",
    "get",
    "list",
    "patch",
    "update",
    "watch"
  ],
  "shortNames": [
    "po"
  ],
  "categories": [
    "all"
  ],
  "storageVersionHash": "xPOwRZ+Yhw8="
}
```

```bash
# Output omitted
```

---

## References

* [Kubernetes API Concepts (official site)](https://kubernetes.io/docs/reference/using-api/api-concepts/)
