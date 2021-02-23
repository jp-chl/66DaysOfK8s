# Day 43 of #66DaysOfK8s

_Last update: 2021-02-22_

---
Today, I have worked with [popeye](https://github.com/derailed/popeye), a command-line application that scans a Kubernetes cluster and reports potentials issues with deployed resources and configurations.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0
* popeye: 0.9.0

---

## Setup

I have installed the brew version: ```brew install derailed/popeye/popeye```.

---

## Configuration

Pass a yaml file as a parameter to run with a specific configuration and also to decide which resources to exclude.

A default run (no params) resources such as nodes, namespaces, pod, deployments, services, etc. It finds features such as CPU/MEM utilization, liveness/readiness missed, ServiceAccount issues, matching pods labels, unused config/secret keys, and so on; Check the full list in [this link](https://github.com/derailed/popeye#sanitizers).

You can also run a specific kubeconfig context with: ```popeye --context <context-name>```.

---

## Usage

For example, scan minikube cluster.

```bash
$ popeye --context minikube

# Output omitted

GENERAL [MINIKUBE]
┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅
  · Connectivity...................................................................................✅
  · MetricServer...................................................................................💥


CLUSTER (1 SCANNED)                                                          💥 0 😱 0 🔊 0 ✅ 1 100٪
┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅
  · Version........................................................................................✅
    ✅ [POP-406] K8s version OK.


CLUSTERROLES (59 SCANNED)                                                  💥 0 😱 0 🔊 15 ✅ 44 100٪
┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅

# Output omitted


PODS (7 SCANNED)                                                               💥 2 😱 5 🔊 0 ✅ 0 0٪
┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅
  · kube-system/coredns-f9fd979d6-57tdp............................................................😱
    🔊 [POP-206] No PodDisruptionBudget defined.
    😱 [POP-301] Connects to API Server? ServiceAccount token is mounted.
    😱 [POP-302] Pod could be running as root user. Check SecurityContext/Image.
    🐳 coredns
      🔊 [POP-105] Liveness probe uses a port#, prefer a named port.
      🔊 [POP-105] Readiness probe uses a port#, prefer a named port.
      😱 [POP-306] Container could be running as root user. Check SecurityContext/Image.
  · kube-system/etcd-minikube......................................................................💥
    🔊 [POP-206] No PodDisruptionBudget defined.
    😱 [POP-301] Connects to API Server? ServiceAccount token is mounted.
    😱 [POP-302] Pod could be running as root user. Check SecurityContext/Image.
    🐳 etcd
      💥 [POP-204] Pod is not ready [0/1].
      😱 [POP-106] No resources requests/limits defined.
      🔊 [POP-105] Liveness probe uses a port#, prefer a named port.
      😱 [POP-104] No readiness probe.
      😱 [POP-306] Container could be running as root user. Check SecurityContext/Image.
```

---

## Sample screenshots

![Official github img example 1](https://raw.githubusercontent.com/derailed/popeye/master/assets/d_score.png)

![Official github img example 2](https://raw.githubusercontent.com/derailed/popeye/master/assets/a_score.png)


---

## References

* [Popeye](https://github.com/derailed/popeye)

* [A curated list of Kubernetes tools](https://collabnix.github.io/kubetools/)
