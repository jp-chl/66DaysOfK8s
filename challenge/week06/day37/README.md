# Day 37 of #66DaysOfK8s

_Last update: 2021-02-16_

---
Today, I have worked with static Pods.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client
* kubectl client and server v1.19.0

---

## Setup

* K8s cluster already created in GCP from scratch (check the instructions in this [link](../../week01/day5/README.md)), or any cluster v 1.19 with at least two worker nodes.
* Set an alias for kubectl (```alias k=kubectl```).
* All Pods run a nginx image.

---

## Tasks

* Identify kubelet process and its config file in both nodes.
* Locate the static Pods manifests directory.
* Deploy static Pods in both nodes.

---

### Identify kubelet process and its config file in both nodes

Pods can be created without using the API Server, but with the ```kubelet daemon```. Normally, [master node components](../../week02/day9)) such as the API Server, the Controller manager and the etcd database run as static Pods.

The kubelet process periodically scans manifests in a folder. Any manifest placed there must be deployed as a static Pod.

Let's identify the running ```kubelet``` process:

```bash
$ student@master: ps -aux|grep kubelet
root      1142  3.4  1.3 1968188 102292 ?      Ssl  00:02   2:51 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2 --resolv-conf=/run/systemd/resolve/resolv.conf
# Output omitted
```

---

### Locate the static pods manifests directory

Within kubelet parameters, a config file can be found (in this case ```--config=/var/lib/kubelet/config.yaml```).

Look for staticPodPath line in the yaml config file:

```bash
$ student@master: sudo grep static /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
```

Based on last command output, any file located in ```/etc/kubernetes/manifests``` hold manifests to be deployed as static Pods.

```bash
$ student@master: sudo ls -l /etc/kubernetes/manifests
total 16
-rw------- 1 root root 2066 Jan 28 22:25 etcd.yaml
-rw------- 1 root root 3648 Jan 28 22:26 kube-apiserver.yaml
-rw------- 1 root root 3346 Jan 28 22:26 kube-controller-manager.yaml
-rw------- 1 root root 1384 Jan 28 22:26 kube-scheduler.yaml
```

Even though the static Pods are managed by the kubelet process, the API Server is also informed about them.

```bash
$ student@master: k get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-7dbc97f587-l74zv   1/1     Running   10         19d
kube-system   calico-node-4z6hn                          1/1     Running   19         31d
kube-system   calico-node-dnwz9                          1/1     Running   15         30d
kube-system   coredns-f9fd979d6-pfdnh                    1/1     Running   10         19d
kube-system   coredns-f9fd979d6-z2llc                    1/1     Running   10         19d
kube-system   etcd-master                                1/1     Running   10         19d
kube-system   kube-apiserver-master                      1/1     Running   10         19d
kube-system   kube-controller-manager-master             1/1     Running   10         19d
kube-system   kube-proxy-2l8pw                           1/1     Running   10         19d
kube-system   kube-proxy-9pw54                           1/1     Running   10         19d
kube-system   kube-scheduler-master                      1/1     Running   10         19d
```

The manifests managed by kubelet are deployed as static Pods with the _node name_ as the suffix. In this example ```-master```.

```bash
$ student@master: k get pods -A -o wide |grep "\-master"
kube-system   etcd-master                                1/1     Running   10         19d   10.2.0.3         master   <none>           <none>
kube-system   kube-apiserver-master                      1/1     Running   10         19d   10.2.0.3         master   <none>           <none>
kube-system   kube-controller-manager-master             1/1     Running   10         19d   10.2.0.3         master   <none>           <none>
kube-system   kube-scheduler-master                      1/1     Running   10         19d   10.2.0.3         master   <none>           <none>
```

---

The same can be done in another node.

```bash
$ student@worker: ps -aux|grep kubelet|grep config
root      1230  1.9  1.2 1976128 96336 ?       Ssl  00:02   1:50 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2 --resolv-conf=/run/systemd/resolve/resolv.conf
```

```bash
$ student@worker: sudo grep static /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
```

```bash
$ student@worker: sudo ls -l /etc/kubernetes/manifests
total 0
```

As you can see in this example, no static Pods are running on the worker node.

---

### Deploy static Pods in both nodes

Let's deploy a simple static Pod, based on nginx image, on each node.

```bash
$ student@master: k run --restart=Never --image=nginx static-nginx --dry-run=client -o yaml > ./static-nginx.yaml
```

```bash
$ student@master: cat static-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: static-nginx
  name: static-nginx
spec:
  containers:
  - image: nginx
    name: static-nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
$ student@master: sudo cp ./static-nginx.yaml /etc/kubernetes/manifests/
```

A new static Pod will be created shortly.

```bash
$ student@master: k get pods -A |grep "\-master"
default       static-nginx-master                        1/1     Running   0          5s
kube-system   etcd-master                                1/1     Running   10         19d
kube-system   kube-apiserver-master                      1/1     Running   10         19d
kube-system   kube-controller-manager-master             1/1     Running   10         19d
kube-system   kube-scheduler-master                      1/1     Running   10         19d
```

If you delete the Pod, with kubectl for example, it will be spawned again by kubelet because the manifest is still located in its config folder.

```bash
$ student@master: k delete pod static-nginx-master
pod "static-nginx-master" deleted
```

```bash
$ student@master: k get pods -A |grep "\-master"
default       static-nginx-master                        1/1     Running   0          10s
kube-system   etcd-master                                1/1     Running   10         19d
kube-system   kube-apiserver-master                      1/1     Running   10         19d
kube-system   kube-controller-manager-master             1/1     Running   10         19d
kube-system   kube-scheduler-master                      1/1     Running   10         19d
```

In order to remove the Pod, just delete the file in the config folder.

```bash
$ student@master: sudo rm /etc/kubernetes/manifests/static-nginx.yaml
```

```bash
$ student@master: k get pods -A |grep "\-master"
kube-system   etcd-master                                1/1     Running   10         19d
kube-system   kube-apiserver-master                      1/1     Running   10         19d
kube-system   kube-controller-manager-master             1/1     Running   10         19d
kube-system   kube-scheduler-master                      1/1     Running   10         19d
```

---

The same guideline can be followed in the worker node.

```bash
$ student@worker: k run --restart=Never --image=nginx static-nginx --dry-run=client -o yaml > ./static-nginx.yaml
```

```bash
$ student@worker: sudo cp ./static-nginx.yaml /etc/kubernetes/manifests/
```

```bash
$ student@master: k get pods -A |grep "\-worker"
default       static-nginx-worker                        1/1     Running   0          6s
```

Remove the Pod:

```bash
$ student@worker: sudo rm /etc/kubernetes/manifests/static-nginx.yaml
```

```bash
$ student@master: k get pods -A |grep "\-worker"
```

---

## References

* [Static Pod (official site)](https://kubernetes.io/docs/concepts/workloads/pods/#static-pods)

* [Create static Pods (official site)](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
