# Day 38 of #66DaysOfK8s

_Last update: 2021-02-17_

---
Today, I have read about the impact of Docker deprecation in K8s.

#kubernetes #learning #K8s #66DaysChallenge

---

## Takeaways

* Docker images will keep working in Kubernetes because they are not really a Docker/specific image, but an OCI ([Open Container Initiative](https://opencontainers.org/)).

* Docker as an underlying runtime is being deprecated, as of v1.20, in favor of runtimes that use the CRI. The Kubelet process manages containers through a Container runtime, for example [containerd](https://containerd.io/) (CNCF graduated project). However, if Docker is used, between containers and Kubelet it is necessary to add [Dockershim](https://github.com/kubernetes/kubernetes/tree/master/pkg/kubelet/dockershim) and a [Docker engine](https://docs.docker.com/engine) because Docker is not compliant with CRI ([Container Runtime Interface](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/)).

* If a managed K8s service is used, a CRI supported version (like containerd or [cri-o](https://cri-o.io/)) is recommended (AKS default is containerd) as soon as possible because as of v1.22 Docker support is removed.

* If "_docker in docker_" pattern is used, it is advisable to migrate to alternatives such as [kaniko](https://github.com/GoogleContainerTools/kaniko), [img](https://github.com/genuinetools/img) or [buildah](https://github.com/containers/buildah).

---

## References

* [Medium article: "Kubernetes is deprecating Docker"](https://medium.com/better-programming/kubernetes-is-deprecating-docker-8a9f7566fbca)

* [Don't Panic: Kubernetes and Docker (official site)](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/)

* [Medium article: "Kubernetes is deprecating Docker in the upcoming release"](https://towardsdatascience.com/kubernetes-is-deprecating-docker-in-the-upcoming-release-2a03d607934a)
