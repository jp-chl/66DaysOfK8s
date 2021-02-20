# Day 40 of #66DaysOfK8s

_Last update: 2021-02-19_

---
Today, I have read about the Service Mesh pattern.

#kubernetes #learning #K8s #66DaysChallenge

---

## Takeaways

* A Service Mesh handles all communication between microservices and implements several cross-cutting concerns such as External configuration, Logging, Health checks, Metrics and Distributed tracing.

* A S.M. implements a proxy application, which it is deployed as a container in the same host (or Kubernetes pod). This proxy (aka sidecar proxy) transparently intercepts the traffic and implements all S.M. features.

* The most popular implementations of S.M. are [Istio](https://landscape.cncf.io/?selected=istio), [Linkerd](https://landscape.cncf.io/?selected=linkerd) and [Consul](https://landscape.cncf.io/?selected=consul). All of them manages, out of the box, auto-proxy injection and TLS. Istio has several features such as traffic redirection (blue/green deployment), traffic splitting (canary deployment). Istio injects [envoy](https://www.cncf.io/projects/) (a CNCF graduated project) as the sidecar proxy.

---

## References

* [Pattern: Service mesh](https://microservices.io/patterns/deployment/service-mesh.html)

* [Kubernetes Service Mesh: A Comparison of Istio, Linkerd and Consul](https://platform9.com/blog/kubernetes-service-mesh-a-comparison-of-istio-linkerd-and-consul/)

* [Service Mesh Is Still Hard (CNCF blog)](https://www.cncf.io/blog/2020/10/26/service-mesh-is-still-hard/)

* [Istio Service Mesh in 2020 (CNCF blog)](https://www.cncf.io/blog/2020/05/25/istio-service-mesh-in-2020/)

* [Istio simple explained in 15 mins](https://www.youtube.com/watch?v=16fgzklcF7Y)
