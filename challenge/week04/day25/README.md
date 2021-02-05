# Day 25 of #66DaysOfK8s

_Last update: 2021-02-04_

---

Today, I have worked with Jobs and CronJobs.

> _Based on: [https://dev.to/itnext/tutorial-basics-of-kubernetes-job-and-cronjob-5c9p](https://dev.to/itnext/tutorial-basics-of-kubernetes-job-and-cronjob-5c9p)_

#kubernetes #learning #K8s #66DaysChallenge

---

## TL;DR

A Job is a process that runs Pods until a specified number of them terminate. A Job keeps track of successful completions.

A CronJob schedule Jobs. Its manifest is similar to a Job one but it has a _"schedule"_ keyword that can be used in linux cron syntax.

[Demo](#demo)

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Setup

* All tests run on minikube.
* All jobs run a busybox image container.

---

## Tasks

* Test examples of Jobs.
* Test examples of CronJobs.

---

## Test examples of Jobs

A Job is a process that runs Pods until a specified number of them terminate. A Job keeps track of successful completions.

A Job manifest is similar to a Pod one.

Let's create a Job that sleeps 90 seconds until it completes.

```yaml
# job1.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job1
spec:
  template:
    spec:
      containers:
        - name: job
          image: busybox
          args:
            - /bin/sh
            - -c
            - date; echo sleeping....; sleep 90s; echo exiting...; date
      restartPolicy: Never
```

```bash
$ kubectl apply -f yaml/job1.yaml
job.batch/job1 created
```

```bash
# Pod status is "Completed"
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job1   0/1           43s        43s

NAME             READY   STATUS    RESTARTS   AGE
pod/job1-gq8z4   1/1     Running   0          43s
```

```bash
$ kubectl logs pod/job1-gq8z4
Fri Feb  5 01:12:34 UTC 2021
sleeping....
exiting...
Fri Feb  5 01:14:04 UTC 2021
```

After 90 seconds, the Job has finished.

```bash
# Notice Pod status is "Completed"
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job1   1/1           96s        2m17s

NAME             READY   STATUS      RESTARTS   AGE
pod/job1-gq8z4   0/1     Completed   0          2m17s
```

```bash
# Cleanup
$ kubectl delete -f yaml/job1.yaml
job.batch "job1" deleted
```

---

You can also specify a deadline limit for the Job to be completed with ```activeDeadlineSeconds``` keyword.

In the following example, the Job's container will be completed after 10 seconds, however, the Job expects to be completed after 5. Therefore, the Job will fail.

```yaml
# job2.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job2
spec:
  activeDeadlineSeconds: 5 # New line
  template:
    spec:
      containers:
        - name: job
          image: busybox
          args:
            - /bin/sh
            - -c
            - date; echo sleeping....; sleep 10s; echo exiting...; date
      restartPolicy: Never
```

```bash
$ kubectl apply -f yaml/job2.yaml
job.batch/job2 created
```

```bash
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job2   0/1           2s         2s

NAME             READY   STATUS              RESTARTS   AGE
pod/job2-gqf95   0/1     ContainerCreating   0          2s
```

```bash
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job2   0/1           7s         7s

NAME             READY   STATUS        RESTARTS   AGE
pod/job2-jvl2z   1/1     Terminating   0          7s
```

```bash
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job2   0/1           24s        24s
```

```bash
$ kubectl get jobs job2 -o yaml|tail -10
status:
  conditions:
  - lastProbeTime: "2021-02-05T01:27:55Z"
    lastTransitionTime: "2021-02-05T01:27:55Z"
    message: Job was active longer than specified deadline
    reason: DeadlineExceeded
    status: "True"
    type: Failed
  failed: 1
  startTime: "2021-02-05T01:27:50Z"
```

```bash
# Cleanup
$ kubectl delete -f yaml/job2.yaml
job.batch "job2" deleted
```

---

There are two more useful keywords that can be used in a Job manifest: ```restartPolicy``` and ```backoffLimit```. The first one to specify what has to happen if the container fails, and the second one to define how many times K8s will try until set the Job status as Failed.

In the following example, the container will fail after 5 seconds (```exit 1```) so K8s the Job will retry (```restartPolicy: OnFailure```) but only twice (```backoffLimit: 2```).

```yaml
# job3.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job3
spec:
  backoffLimit: 2 # New line
  template:
    spec:
      containers:
        - name: job
          image: busybox
          args:
            - /bin/sh
            - -c
            - date; echo sleeping....; sleep 5s; exit 1; # exit 1: Fail status
      restartPolicy: OnFailure # New line
```

```bash
$ kubectl apply -f yaml/job3.yaml
job.batch/job3 created
```

```bash
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job3   0/1           2s         2s

NAME             READY   STATUS              RESTARTS   AGE
pod/job3-tx755   0/1     ContainerCreating   0          2s
```

```bash
# First run
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job3   0/1           5s         5s

NAME             READY   STATUS    RESTARTS   AGE
pod/job3-2tfjg   1/1     Running   0          5s
```

```bash
# First failure
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job3   0/1           13s        13s

NAME             READY   STATUS   RESTARTS   AGE
pod/job3-tx755   0/1     Error    0          13s
```

```bash
# Second try, 1 restart
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job3   0/1           17s        17s

NAME             READY   STATUS    RESTARTS   AGE
pod/job3-tx755   1/1     Running   1          17s
```

```bash
# Second failure
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job3   0/1           23s        23s

NAME             READY   STATUS   RESTARTS   AGE
pod/job3-tx755   0/1     Error    1          23s
```

```bash
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job3   0/1           29s        29s

NAME             READY   STATUS             RESTARTS   AGE
pod/job3-tx755   0/1     CrashLoopBackOff   1          29s
```

```bash
# Second error, terminating Pod
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job3   0/1           35s        35s

NAME             READY   STATUS        RESTARTS   AGE
pod/job3-tx755   1/1     Terminating   2          35s
```

```bash
$ kubectl get jobs,pods
NAME             COMPLETIONS   DURATION   AGE
job.batch/job3   0/1           44s        44s
```

```bash
# Cleanup
$ kubectl delete -f yaml/job3.yaml
job.batch "job3" deleted
```

---

## Test examples of CronJobs

> _This feature is in beta._

A CronJob schedule Jobs. Its manifest is similar to a Job one but it has a ```schedule``` keyword that can be used in linux cron syntax ([check this link](https://en.wikipedia.org/wiki/Cron)).

The next example repeats a Job every minute. The Job's container sleeps for 5 seconds.

```yaml
# cronjob1.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob1
spec:
  schedule: "*/1 * * * *" # every 1 minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: cronjob
              image: busybox
              args:
                - /bin/sh
                - -c
                - date; echo sleeping....; sleep 5s; echo exiting...;
          restartPolicy: Never
```

```bash
$ kubectl apply -f yaml/cronjob1.yaml
cronjob.batch/cronjob1 created
```

```bash
$ kubectl get cronjob,job,pod
NAME                     SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/cronjob1   */1 * * * *   False     0        <none>          2s
```

```bash
# The CronJob spawns a Job, and the latter a Pod
$ kubectl get cronjob,job,pod
NAME                     SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/cronjob1   */1 * * * *   False     1        9s              29s

NAME                            COMPLETIONS   DURATION   AGE
job.batch/cronjob1-1612490940   0/1           8s         8s

NAME                            READY   STATUS    RESTARTS   AGE
pod/cronjob1-1612490940-jthpx   1/1     Running   0          8s
```

```bash
$ kubectl get cronjob,job,pod
NAME                     SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/cronjob1   */1 * * * *   False     1        17s             37s

NAME                            COMPLETIONS   DURATION   AGE
job.batch/cronjob1-1612490940   1/1           10s        16s

NAME                            READY   STATUS      RESTARTS   AGE
pod/cronjob1-1612490940-jthpx   0/1     Completed   0          16s
```

```bash
# After 1 minute, a second Job runs
$ kubectl get cronjob,job,pod
NAME                     SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/cronjob1   */1 * * * *   False     1        3s              83s

NAME                            COMPLETIONS   DURATION   AGE
job.batch/cronjob1-1612490940   1/1           10s        62s
job.batch/cronjob1-1612491000   0/1           2s         2s

NAME                            READY   STATUS              RESTARTS   AGE
pod/cronjob1-1612490940-jthpx   0/1     Completed           0          62s
pod/cronjob1-1612491000-6ncxx   0/1     ContainerCreating   0          2s
```

```bash
$ kubectl get cronjob,job,pod
NAME                     SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/cronjob1   */1 * * * *   False     1        8s              2m28s

NAME                            COMPLETIONS   DURATION   AGE
job.batch/cronjob1-1612490940   1/1           10s        2m7s
job.batch/cronjob1-1612491000   1/1           10s        67s
job.batch/cronjob1-1612491060   0/1           7s         7s

NAME                            READY   STATUS      RESTARTS   AGE
pod/cronjob1-1612490940-jthpx   0/1     Completed   0          2m7s
pod/cronjob1-1612491000-6ncxx   0/1     Completed   0          67s
pod/cronjob1-1612491060-tlp62   1/1     Running     0          7s
```

```bash
# Cleanup
$ kubectl delete -f yaml/cronjob1.yaml
cronjob.batch "cronjob1" deleted
```

---

## References

* [Jobs (official site)](https://kubernetes.io/docs/concepts/workloads/controllers/job/)

* [CronJob (official site)](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
