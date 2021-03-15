# Day 63 of #66DaysOfK8s

_Last update: 2021-03-14_

---
Today, in part 2, I have continued to practice with CKAD exercises.

#kubernetes #learning #K8s #66DaysChallenge

---

## Takeaways

* First part in [this link](../day62).

* In my second attempt, I've had to work with [Node affinity](https://github.com/jp-chl/66DaysOfK8s/blob/master/challenge/week06/day36), [Taints and tolerations](https://github.com/jp-chl/66DaysOfK8s/blob/master/challenge/week05/day35), [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/), [PV](https://github.com/jp-chl/66DaysOfK8s/blob/master/challenge/week05/day30), [Jobs](https://github.com/jp-chl/66DaysOfK8s/blob/master/challenge/week04/day25).

* A very useful command is ```k explain <kubernetes object> --recursive```, where you can have a yaml template ready to use. For instance, Ingress is explained like:

```bash
$ k explain ingress --recursive
KIND:     Ingress
VERSION:  extensions/v1beta1

DESCRIPTION:
     Ingress is a collection of rules that allow inbound connections to reach
     the endpoints defined by a backend. An Ingress can be configured to give
     services externally-reachable urls, load balance traffic, terminate SSL,
     offer name based virtual hosting etc. DEPRECATED - This group version of
     Ingress is deprecated by networking.k8s.io/v1beta1 Ingress. See the release
     notes for more information.

FIELDS:
   apiVersion   <string>
   kind <string>
   metadata     <Object>
      annotations       <map[string]string>
      clusterName       <string>
      creationTimestamp <string>
      deletionGracePeriodSeconds        <integer>
      deletionTimestamp <string>
      finalizers        <[]string>
      generateName      <string>
      generation        <integer>
      labels    <map[string]string>
      managedFields     <[]Object>
         apiVersion     <string>
         fieldsType     <string>
         fieldsV1       <map[string]>
         manager        <string>
         operation      <string>
         time   <string>
# Output omitted
```

* It is advisable to add ```--dry-run``` and the end of an imperative command in order to test whether the output will be valid.

---

## References

* [Kubernetes Journey — CKA / CKAD Exam Tips (ITNext article by Brad McCoy)](https://itnext.io/kubernetes-journey-cka-ckad-exam-tips-ff73e4672833)

* [CKAS exercises (Github article by Dimitris-Ilias Gkanatsios)](https://github.com/dgkanatsios/CKAD-exercises)
