# Day 64 of #66DaysOfK8s

_Last update: 2021-03-15_

---
Today, in part 3, I have continued to practice with CKAD exercises.

#kubernetes #learning #K8s #66DaysChallenge

---

## Takeaways

* First part in [this link](../day62).

* In my third attempt, I've had to work with [Taints and tolerations](https://github.com/jp-chl/66DaysOfK8s/blob/master/challenge/week05/day35), [Network policies](https://github.com/jp-chl/66DaysOfK8s/blob/master/challenge/week05/day32), [executing commands in Pods](), and [mounting volumes in Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/) (either from an [emptyDir](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/) or [from a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)).

* In my case, I've had some issues by attempting to run a command within a Pod. Normally, most of the container public images have already installed ```sh```, so a container command should look like this:

```yaml
# pod
spec:

  containers:
  - image: busybox
    name: my-pod
    command: [ "/bin/sh", "-c", "date" ]
```

* As you have access to the official Kubernetes documentation, it is useful to have the Cheat Sheet at hand (available at this [link](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)).

---

## References

* [Kubernetes Journey — CKA / CKAD Exam Tips (ITNext article by Brad McCoy)](https://itnext.io/kubernetes-journey-cka-ckad-exam-tips-ff73e4672833)

* [CKAS exercises (Github article by Dimitris-Ilias Gkanatsios)](https://github.com/dgkanatsios/CKAD-exercises)
