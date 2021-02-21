# Day 41 of #66DaysOfK8s

_Last update: 2021-02-20_

---
Today, I have read about differences between EKS, GKE and AKS.

> _Based on [StackRox article by Michael Foster](https://www.stackrox.com/post/2021/01/eks-vs-gke-vs-aks-jan2021/)_

#kubernetes #learning #K8s #66DaysChallenge

---

## Takeaways

* This comparison cover concepts such as version availability, network and security options, and container image services.

* Control-plane upgrade process: EKS user must manually update the system services that run on nodes (e.g. kube-proxy), but AKS and GKE system components are updated with cluster upgrades.

* Node upgrade process: EKS user have to explicitly initiate upgrades either with unmanaged or managed node groups, but AKS and GKE are automatically upgraded.

* Node OS: EKS uses Amazon Linux 2, AKS and GKE Ubuntu.

* Container runtime: All of them use Docker by default, and they support containerd.

* Control plane high HA options: EKS c.p. is deployed across multiple availability zones by default, AKS to the number of zones defined by the Admin, and GKE a single control planes but three in regional clusters case.

* Pricing: EKS and GKE charge $0.10/hour and AKS pay as you go.

---

## References

* [EKS vs GKE vs AKS - Evaluating Kubernetes in the Cloud](https://www.stackrox.com/post/2021/01/eks-vs-gke-vs-aks-jan2021/)
