# Day 30 of #66DaysOfK8s

_Last update: 2021-02-09_

---
Today, I have worked with a simple example of Persistent volume and Persistent volume claim usage.

> _Based on: [Demo from official site](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)_

#kubernetes #learning #K8s #66DaysChallenge

---

## TL;DR

A _PersistentVolume_ (PV) is a piece of storage that Pods can use via a _PersistentVolumeClaim_ (PVC). The PV lifecycle is independent of the Pods, and the PV API object handles the specific storage implementation (NFS, iSCSI, etc.).

A _PersistentVolumeClaim_ (PVC) is a request for storage made by Pods. Specific size and [access modes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) can be requested by Pods.

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

---

## Tasks

* Create a path to be used as a volume in the worker node.
* Create a Persistent volume linked to the path created in the worker node.
* Create a Persistent volume claim linked to the Persistent volume.
* Create a Pod associated with the Persistent volume claim.

---

### Create a path to be used as a volume in the worker node

Since minikube is used, ssh to the the worker node.

```bash
$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)
```

```bash
# Not logged as root
$ id
uid=1000(docker) gid=1000(docker) groups=1000(docker),10(wheel),1018(vboxsf)
```

Create a "```data```" directory within ```/mnt```.

```bash
$ sudo mkdir /mnt/data
```

Create a simple index.html in the previously created directory.

```bash
$ sudo sh -c "echo 'Hello from K8s storage' > /mnt/data/index.html"
```

```bash
$ cat /mnt/data/index.html
Hello from K8s storage
```

Go back to the terminal.

```bash
$ exit
logout
```

---

## Create a Persistent volume linked to the path created in the worker node

```yaml
# pv-volume.yaml
apiVersion:
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual # string to be mapped with PVC
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data" # Directory created in the worker node
```

```bash
$ kubectl apply -f yaml/pv-volume.yaml
persistentvolume/task-pv-volume created
```

```bash
$ kubectl get pv
NAME                              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
persistentvolume/task-pv-volume   10Gi       RWO            Retain           Available           manual                  10
```

---

## Create a Persistent volume claim linked to the Persistent volume

```yaml
# pv-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual # string to be mapped with PV
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

```bash
$ kubectl apply -f yaml/pv-claim.yaml
persistentvolumeclaim/task-pv-claim created
```


Notice what happens after PV and PVC are linked to each other.

```bash
$ kubectl get pvc task-pv-claim -o jsonpath='{.spec.volumeName}'
task-pv-volume
```

```bash
$ kubectl get pv task-pv-volume -o jsonpath='{.spec.claimRef.name}'
task-pv-claim
```

---

## Create a Pod associated with the Persistent volume claim

Now a Pod can use the storage by claiming it.

```yaml
# pv-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim # PVC name
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

```bash
$ kubectl apply -f yaml/pv-pod.yaml
pod/task-pv-pod created
```

```bash
$ kubectl get pods task-pv-pod -o jsonpath='{.spec.volumes[0].persistentVolumeClaim.claimName}'
task-pv-claim
```

---

Now, access to index.html can be tested by doing curl to localhost (nginx image) within the pod.

```bash
$ kubectl exec -ti pod/task-pv-pod -- sh
```

```bash
$ apt update
Get:1 http://deb.debian.org/debian buster InRelease [122 kB]
# Output omitted
Reading state information... Done
All packages are up to date.
```

```bash
$ apt install curl
Reading package lists... Done
# Output omitted
curl is already the newest version (7.64.0-4+deb10u1).
```

```bash
$ curl http://localhost
Hello from K8s storage
```

```bash
$ exit
```

---

### Cleanup

```bash
$ kubectl delete -f yaml/.
persistentvolumeclaim "task-pv-claim" deleted
pod "task-pv-pod" deleted
persistentvolume "task-pv-volume" deleted
```

---

## References

* [Persistent volumes (official site)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

---

# Demo

[![asciicast](https://asciinema.org/a/7s6PQuJLSZQ5n6exuxDvJkXuX.svg)](https://asciinema.org/a/7s6PQuJLSZQ5n6exuxDvJkXuX)
