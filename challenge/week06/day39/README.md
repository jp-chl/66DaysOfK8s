# Day 39 of #66DaysOfK8s

_Last update: 2021-02-18_

---
Today, I have read about the Operator pattern.

#kubernetes #learning #K8s #66DaysChallenge

---

## Takeaways

* Operators are software extensions that take advantage of Kubernetes extensibility. It uses [custom resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) and follows [control loop](https://kubernetes.io/docs/concepts/architecture/controller) principles.

* Operators automate workloads, for example deployments but you can control how Kubernetes does it. Operators are clients of the Kubernetes API, and act as controllers for a custom resource.

* For example, an Operator can restore DB backups, upgrade applications or inject a failure in a cluster.

* Usually, an Operator is deployed by adding a Custom Resource Definition and its associated Controller, but running outside of the [control plane](../../week02/day9) (e.g. a deployment).

* You can write your own Operator using any Kubernetes API client (for example with Go Language).

---

## References

* [Operator pattern (Kubernetes official site)](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)

* [Operators (Red Hat website)](https://www.openshift.com/learn/topics/operators)

* [OperatorHub.io (community shared Operators)](https://operatorhub.io/)

* [Operator Framework](https://operatorframework.io/)
