# Day 59 of #66DaysOfK8s

_Last update: 2021-03-10_

---
Today I have worked with Stateful Sets.

#kubernetes #learning #K8s #66DaysChallenge

---

## Takeaways

* StatefulSets is the workload API object used to manage stateful applications.

* It's similar to a Deployment, but a StatefulSet keeps track of its Pod's identities in an ordered way (e.g. like pod-0, pod-1, etc.).

* A StatefulSet links every Pod to a PV, and after a Pod is destroyed is linked again to the same PV; A StatefulSet deletion does not trigger PVC/PV elimination (it has to be done manually).

* A common use case is when you need a DB cluster where a master should be always identifiable (for instance, a Pod could be named as mysql-0).

* Pod's creation and deletion is managed in an ordered manner. In a three replica example, a second Pod is created only after the first one is running, a third after the second, and so on. Deletion is done in the opposite direction, e.g., the second is deleted after the third, the first after the second.

* A network identity Pod discovery is done with a [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services), which has to be created manually.

A typical StatefulSet example is like the following:

```yaml
# StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sts-example
spec:
  replicas: 3
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: stateful
      v: v1
  serviceName: app
  updateStrategy:
    type: OnDelete
  template:
    metadata:
      labels:
        app: stateful
        v: v1
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: standard
      resources:
        requests:
          storage: 1Gi
```

The last yaml spawns 3 Pods, 3 PVC and 3 PV.

```bash
$ kubectl apply -f yaml/ss.yaml
statefulset.apps/sts-example created
```

A pod called ```sts-example-0``` will be created. Then ```sts-example-1``` and lastly ```sts-example-2```.

```bash
$ kubectl get pv,pvc,pod,statefulset
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   REASON   AGE
persistentvolume/pvc-fb1f8e93-14ea-4aa6-9c86-b92350639af4   1Gi        RWO            Delete           Bound    default/www-sts-example-0   standard                8s

NAME                                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/www-sts-example-0   Bound    pvc-fb1f8e93-14ea-4aa6-9c86-b92350639af4   1Gi        RWO            standard       9s

NAME                READY   STATUS              RESTARTS   AGE
pod/sts-example-0   0/1     ContainerCreating   0          8s

NAME                           READY   AGE
statefulset.apps/sts-example   0/3     9s
```

```bash
$ kubectl get pv,pvc,pod,statefulset
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   REASON   AGE
persistentvolume/pvc-5f8eb310-7c1f-4d19-b82b-0671d2bd1c45   1Gi        RWO            Delete           Bound    default/www-sts-example-2   standard                64s
persistentvolume/pvc-f65ae4d1-c8d0-4d64-8825-4684b24ad223   1Gi        RWO            Delete           Bound    default/www-sts-example-1   standard                68s
persistentvolume/pvc-fb1f8e93-14ea-4aa6-9c86-b92350639af4   1Gi        RWO            Delete           Bound    default/www-sts-example-0   standard                77s

NAME                                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/www-sts-example-0   Bound    pvc-fb1f8e93-14ea-4aa6-9c86-b92350639af4   1Gi        RWO            standard       78s
persistentvolumeclaim/www-sts-example-1   Bound    pvc-f65ae4d1-c8d0-4d64-8825-4684b24ad223   1Gi        RWO            standard       68s
persistentvolumeclaim/www-sts-example-2   Bound    pvc-5f8eb310-7c1f-4d19-b82b-0671d2bd1c45   1Gi        RWO            standard       64s

NAME                READY   STATUS    RESTARTS   AGE
pod/sts-example-0   1/1     Running   0          77s
pod/sts-example-1   1/1     Running   0          68s
pod/sts-example-2   1/1     Running   0          64s

NAME                           READY   AGE
statefulset.apps/sts-example   3/3     79s
```

---

## References

* [StatefulSets (official site)](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
