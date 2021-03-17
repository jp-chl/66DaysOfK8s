# Day 65 of #66DaysOfK8s

_Last update: 2021-03-16_

---
Today, in part 4, I have continued to practice with CKAD exercises.

#kubernetes #learning #K8s #66DaysChallenge

---

> _First part in [this link](https://github.com/jp-chl/66DaysOfK8s/blob/master/challenge/week09/day62)_

## Takeaways

* In my fourth attempt, I've had to work with [liveness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-command) and readiness probes, [Cronjob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/), accessing [Secrets as mounted volumes](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod), and [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/).

* Some questions force you to pay special attention to matching http ports (e.g. containerPort vs liveness http port).

* Besides the very useful [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/), there is also an excellent Kubernetes documentation reference focused on **imperative commands** (available in this [link](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)).

---

## References

* [Kubectl imperative commands guide (official site)](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

* [Kubernetes Cheat Sheet (official site)](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

* [Kubernetes Journey — CKA / CKAD Exam Tips (ITNext article by Brad McCoy)](https://itnext.io/kubernetes-journey-cka-ckad-exam-tips-ff73e4672833)

* [CKAS exercises (Github article by Dimitris-Ilias Gkanatsios)](https://github.com/dgkanatsios/CKAD-exercises)
