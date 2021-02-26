# Day 46 of #66DaysOfK8s

_Last update: 2021-02-25_

---
Today, I have read about Ingress controller.

> _Based on [Medium article by Ishan Liyanage](https://ishanul.medium.com/what-is-ingress-in-kubernetes-b66e737b4678)_

#kubernetes #learning #K8s #66DaysChallenge

---

## Takeaways

* Ingress makes an application accessible via a single external URL that can be configured to route to different services in the cluster based on the URL part.

* It can implement SSL.

* Ingress controllers are not started automatically with a cluster.

* It can be configured from a native K8s manifest.

* It's similar to layer 7 K8s load balancer.

* There are several implementations such as [Contour](https://projectcontour.io/), [Gloo](https://gloo.solo.io/), [Istio](https://istio.io/), [Nginx](https://www.nginx.com/products/nginx-ingress-controller/), etc.

---

## References

* [Ingress Controllers (official site)](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
