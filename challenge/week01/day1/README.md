# Day 1 of #66DaysOfK8s

_Last update: 2021-01-11_

---

Today, I've learned to create a K8s cluster with K0s (and Multipass). It is nice to have an alternative to run a local K8s cluster rather than the typical minikube.

> _Based on: https://medium.com/better-programming/k0s-kubernetes-in-a-single-binary-224bb43f4520_

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* multipass: 1.5.0+mac
* k0s: 0.9.1

---

## Setup

* Install [multipass](https://multipass.run/)

---

## Code

_The following commands create 3 Ubuntu instances on xhyve. Each VM has 5 GB of disk, 2 GB of RAM and 2 vCPUs:_

```bash
for i in 1 2 3; do 
  multipass launch -n node$i -c 2 -m 2G
done
```

> _You will see "Starting Node..."_

You can check the 3 machines created:

```bash
$ multipass list
Name                    State             IPv4             Image
node1                   Running           192.168.64.53    Ubuntu 20.04 LTS
node2                   Running           192.168.64.54    Ubuntu 20.04 LTS
node3                   Running           192.168.64.55    Ubuntu 20.04 LTS
```

Install K0s in every node (it may take a while).

```bash
for i in 1 2 3; do 
  multipass exec node$i -- bash -c "curl -sSLf get.k0s.sh | sudo sh"
done
```

Check k0s version:

```bash
$ multipass exec node1 -- k0s version # As of 2021-01-11: 0.9.1
v0.9.1
```

---

## Create a configuration file

Enter node, 1 in this case:

```bash
$ multipass shell node1
```

Instead of using the default config (```ubuntu@node1:~$ k0s default-config```), we're using a very simple configuration that must be saved in ```/etc/k0s/k0s.yaml```.

```yaml
kind: Cluster
metadata:
  name: k0s
spec:
  api:
    address: 192.168.64.11 # Node IP Address
    sans:
    - 192.168.64.11 # Load balancer whenever additional masters nodes are behind it
  network:
    podCIDR: 10.244.0.0/16
    serviceCIDR: 10.96.0.0/12
```

## Initialization of the Cluster

On ```node 1``` create a [systemd Unit file](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files) to manage k0s

```bash
ubuntu@node1:~$ sudo vi /lib/systemd/system/k0s.service
````

Set its content as:

```
[Unit]
Description="k0s server"
After=network-online.target
Wants=network-online.target
 
[Service]
Type=simple
ExecStart=/usr/bin/k0s server -c /etc/k0s/k0s.yaml --enable-worker
Restart=always
```

> ```ExecStart``` runs the k0s server. ```--enable-worker``` enables the master to also act as a worker.

Reload systemd, and start our newly created service:

```bash
ubuntu@node1:~$ sudo systemctl daemon-reload

ubuntu@node1:~$ sudo systemctl start k0s.service
```

To check k0s processes are running:

```bash
ubuntu@node1:~$ sudo ps aux | awk '{print $11}' | grep k0s
```

```bash
/usr/bin/k0s
/var/lib/k0s/bin/etcd
/var/lib/k0s/bin/konnectivity-server
/var/lib/k0s/bin/kube-controller-manager
/var/lib/k0s/bin/kube-apiserver
/var/lib/k0s/bin/kube-scheduler
/usr/bin/k0s
/var/lib/k0s/bin/containerd
/var/lib/k0s/bin/kubelet
```

> _k0s is in charge of managing all the kubernetes master components._

**A one-node cluster is operative.**

---

## Accessing the Cluster

In node 1, k0s generates a default ```kubeconfig``` during cluster creation in ```/var/lib/k0s/pki/admin.conf```.

Save kubeconfig locally as ```k0s.cfg```:

```bash
# Get kubeconfig file
$ multipass exec node1 -- sudo cat /var/lib/k0s/pki/admin.conf > ./k0s.cfg
```

Replace ```k0s.cfg``` internal IP address with ```node1's``` external one:

```bash
export NODE1_IP=$(multipass info node1 | grep IP | awk '{print $2}')
# Replace localhost with $NODE1_IP
sed -i '' "s/localhost/$NODE1_IP/" ./k0s.cfg
```

Set Kubeconfig as our recent ```k0s.cfg``` file:

```bash
export KUBECONFIG=$PWD/k0s.cfg
```

Check running cluster:

```bash
$ kubectl get nodes
NAME    STATUS   ROLES    AGE   VERSION
node1   Ready    <none>   13m   v1.20.1-k0s1
```

> _```node1``` is also a worker (check ```--enable-worker``` flag above)_

---

## Adding Worker Nodes

Create a ```join token``` from ```node1``` in order ```to add node2 and node3```:

In ```node 1``` (master), run:

```bash
export TOKEN=$(sudo k0s token create --role=worker)
```

Enter node 2 and add to the cluster:

```bash
$ multipass shell node2
ubuntu@node2:~$ sudo k0s worker $TOKEN # Copy token from master
```

Same but with node 3:

```bash
$ multipass shell node3
ubuntu@node3:~$ sudo k0s worker $TOKEN # Copy token from master
```

After a while you will see all nodes available:

```bash
$ kubectl get nodes
NAME    STATUS   ROLES    AGE     VERSION
node1   Ready    <none>   33m     v1.20.1-k0s1
node2   Ready    <none>   88s     v1.20.1-k0s1
node3   Ready    <none>   4m18s   v1.20.1-k0s1
```

Check pods running in all nodes:

```bash
$ kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5f6546844f-ztdvk   1/1     Running   0          34m
kube-system   calico-node-8m6n6                          1/1     Running   0          34m
kube-system   calico-node-cxjbw                          1/1     Running   0          4m54s
kube-system   calico-node-x8f4p                          1/1     Running   0          2m4s
kube-system   coredns-5c98d7d4d8-6tj9f                   1/1     Running   0          34m
kube-system   konnectivity-agent-f6ps5                   1/1     Running   0          33m
kube-system   konnectivity-agent-gwqn7                   1/1     Running   0          83s
kube-system   konnectivity-agent-nrvws                   1/1     Running   0          4m23s
kube-system   kube-proxy-566h7                           1/1     Running   0          34m
kube-system   kube-proxy-shvcp                           1/1     Running   0          2m4s
kube-system   kube-proxy-zvcsj                           1/1     Running   0          4m54s
kube-system   metrics-server-7d4bcb75dd-4hfhk            1/1     Running   0          34m
```

