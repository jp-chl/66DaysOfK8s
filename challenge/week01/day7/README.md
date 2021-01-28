# Day 7 of #66DaysOfK8s

_Last update: 2021-01-17_

---

Today, I have worked in part 3 of a series of lessons in order to create a K8s cluster from scratch in GCP.
In this lesson I have added the worker node to the cluster.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client

---

## Setup

* Master node created ([part 1 link](../day5/README.md))
* K8s software running on master node ([part 2 link](../day6/README.md))
* We'll be installing K8s version 1.18.1 (on the worker node too)
* Many of the steps in the master node applies in the worker one
* _Savings tip_: You can stop VM instances if you're not using them.
* If you, like me, have had issues connecting to the VM instances via SSH (after stop and start them regularly), there is a workaround ([check this link](../../workarounds.md)).

---

## Tasks

* Connect to worker node and install the required software
* On the master node, create a token and a certificate for the worker to join the cluster
* On the worker node, configure kubeadm to join the cluster
* Test installation

---

### Connect to worker node and install the required software

Using the SSK keys obtained in previous parts, connect via ssh to the GCP VM instance associated with the worker node:

```bash
# 1.2.3.4: replace with your own WORKER node public ip
$ ssh -i "K8sPK1" student@1.2.3.4
The authenticity of host '1.2.3.4 (1.2.3.4)' can't be established.
ECDSA key fingerprint is SHA256:aknddjkj2ndbj213ndb23bndoi2ndbnjx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '1.2.3.4' (ECDSA) to the list of known hosts.

...

Last login: Fri Jan 15 11:46:26 2021 from 1.2.3.4
student@worker:~$
```

Log in as root user. Update and upgrade the system

```bash
$ student@worker:~$ sudo -i
root@worker:~#
```

```bash
# First and last command results for clarity
$ root@worker:~$ apt-get update && apt-get upgrade -y
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease

...

0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

Install a text editor (vi, nano, etc.)

```bash
# First and last command results for clarity
$ root@worker:~$ apt-get install -y vim
Reading package lists... Done

...

0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

Install a container environment (docker as in the previous part)

```bash
# Only some command results are shown below
$ root@worker:~$ apt-get install -y docker.io
Reading package lists... Done

...

The following additional packages will be installed:
  bridge-utils cgroupfs-mount containerd pigz runc ubuntu-fan
Suggested packages:
  ifupdown aufs-tools debootstrap docker-doc rinse zfs-fuse | zfsutils


Adding group 'docker' (GID 116) ...
Done.
Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /lib/systemd/system/docker.socket.
docker.service is a disabled or a static unit, not starting it.
Processing triggers for systemd (237-3ubuntu10.43) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for ureadahead (0.100.0-21) ...
```

Add a new repo for K8s

Create a ```kubernetes.list``` file in ```/etc/apt/sources.list.d``` folder and add an entry for the main repo for K8s distribution.

```bash
$ root@worker:~$ vim /etc/apt/sources.list.d/kubernetes.list
```

> _```kubernetes.list``` content_
```bash
deb http://apt.kubernetes.io/ kubernetes-xenial main
```

Add a GPG key for the packages

```bash
$ root@worker:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
OK
```

Update new repo

```bash
# First and last command results for clarity
$ root@worker:~$ apt-get update
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease

...

Reading package lists... Done
```

Install main components (kubeadm, kubelet, kubectl)

If you want to install the newest versions, you can omit the equal sign on the command line. Normally, newest versions might have bugs. In this case we're installing the same version as the master node.

```bash
# Only some command results are shown
$ root@worker:~$ apt-get install -y kubeadm=1.18.1-00 kubelet=1.18.1-00 kubectl=1.18.1-00
Reading package lists... Done

...

Unpacking conntrack (1:1.4.4+snapshot20161117-6ubuntu2) ...
Unpacking cri-tools (1.13.0-01) ...
Unpacking kubernetes-cni (0.8.7-00) ...
Unpacking socat (1.7.3.2-2ubuntu2) ...
Unpacking kubelet (1.18.1-00) ...
Unpacking kubectl (1.18.1-00) ...
Unpacking kubeadm (1.18.1-00) ...

Setting up conntrack ...

...

Created symlink /etc/systemd/system/multi-user.target.wants/kubelet.service → /lib/systemd/system/kubelet.service.

...

Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

Hold the software at the recent but stable version

```bash
$ root@worker:~$ apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
```

---

### Create a token and a certificate for the worker to join the cluster

On the master node, find its IP address. At the moment, the primary GCE interface for this node type is ```ens4```.

```bash
$ root@master:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    ...
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
    link/ether 42:01:0a:02:00:03 brd ff:ff:ff:ff:ff:ff
    inet 10.2.0.3/32 scope global dynamic ens4
       valid_lft 2177sec preferred_lft 2177sec
    inet6 fe80::4001:aff:fe02:3/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    ...
4: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1440 qdisc noqueue state UNKNOWN group default qlen 1000
    ...
7: cali566ba2f164b@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1440 qdisc noqueue state UP group default
    ...
8: cali442b62124a8@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1440 qdisc noqueue state UP group default
    ...
9: calibb3218fb43a@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1440 qdisc noqueue state UP group default
    ...
```

> _In this case, the master node IP is 10.2.0.3_
```bash
$ root@master:~$ ip addr show ens4 | grep inet
    inet 10.2.0.3/32 scope global dynamic ens4
    inet6 fe80::4001:aff:fe02:3/64 scope link
```

List created tokens:

```bash
student@master:~$ sudo kubeadm token list
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
d6ovfk.q9sqidty6s7jp8wu   8h          2021-01-17T23:30:03Z   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
```

Create a token.
> _It will be used later on the worker node, in this case ```2xfb7a.w9kh9vvus4nnb0iz```_
```bash
student@master:~$ sudo kubeadm token create
W0117 14:55:42.999036    3700 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
2xfb7a.w9kh9vvus4nnb0iz                                                   system:bootstrappers:kubeadm:default-node-token
```

Create and use a Discovery Token CA Cert Hash created from the master to ensure the node joins the cluster in a secure manner; _Pay attention to the special characters!_
> _It will be used later on the worker node, in this case ```e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855```_
```bash
student@master:~$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
(stdin)= be5a3ee9b1b2f710f76c389d48ef31753dfe0c2b8c36359c281fdcc2c7eee74b
```

---

### On the worker node, configure kubeadm to join the cluster

Add a local DNS alias for the master server.
Edit the ```/etc/hosts``` file and add the **master** IP address and assign the name k8smaster (node alias).

```bash
$ root@worker:~$ vim /etc/hosts
```

> _```/etc/hosts``` content_
```bash
10.2.0.3 k8smaster # add this line
127.0.0.1 localhost

# ...
```

Use the token and hash obtained above, in this case as sha256:long-hash to join the cluster from the worker node. Use the private IP address of the master server and port 6443.

```bash
$ root@worker:~$ kubeadm join --token 2xfb7a.w9kh9vvus4nnb0iz k8smaster:6443 --discovery-token-ca-cert-hash sha256:be5a3ee9b1b2f710f76c389d48ef31753dfe0c2b8c36359c281fdcc2c7eee74b
W0117 15:17:02.878531   12600 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

---

### Test installation

On the worker, allow not-root admin level access to the cluster.

```bash
$ root@worker:~$ exit
logout
student@worker:~$
```

If you try to run the kubectl command it should fail. You must have a ```.kube/config``` file.

```bash
$ student@worker:~$ kubectl get nodes
error: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
```

```bash
$ student@worker:~$ ls -l .kube
ls: cannot access '.kube': No such file or directory
```

In order to connect to the cluster, create a config file with the same content as the master node (_see previous part_).

```bash
$ student@worker:~$ mkdir -p $HOME/.kube
```

```bash
$ student@worker:~$ vi .kube/config
```

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ... # a long string...
    server: https://k8smaster:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: ...
    client-key-data: ...
```

---

Now, check again.


```bash
$ student@worker:~$ kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.1", GitCommit:"7879fc12a63337efff607952a323df90cdc7a335", GitTreeState:"clean", BuildDate:"2020-04-08T17:38:50Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.1", GitCommit:"7879fc12a63337efff607952a323df90cdc7a335", GitTreeState:"clean", BuildDate:"2020-04-08T17:30:47Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

```bash
$ student@worker:~$ kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
master   Ready    master   15h     v1.18.1
worker   Ready    <none>   6m10s   v1.18.1
```

```bash
$ student@worker:~$ kubectl -n kube-system get pods
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-7dbc97f587-z579v   1/1     Running   1          15h
calico-node-4z6hn                          1/1     Running   1          15h
calico-node-dnwz9                          1/1     Running   0          8m41s # new pod
coredns-66bff467f8-b4vdq                   1/1     Running   1          15h
coredns-66bff467f8-vhmg2                   1/1     Running   1          15h
etcd-master                                1/1     Running   1          15h
kube-apiserver-master                      1/1     Running   1          15h
kube-controller-manager-master             1/1     Running   1          15h
kube-proxy-v8hwr                           1/1     Running   1          15h
kube-proxy-zrcjm                           1/1     Running   0          8m41s # new pod
kube-scheduler-master                      1/1     Running   1          15h
```

---

If you describe the master node you can view its resources and status.

_The master won’t allow non-infrastructure pods by default for security and resource contention reasons (take a look at the status of **[Taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)**)._

```bash
student@master:~$ kubectl describe node master
Name:               master
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=master
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 10.2.0.3/32
                    projectcalico.org/IPv4IPIPTunnelAddr: 192.168.219.64
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Sat, 16 Jan 2021 23:30:00 +0000
Taints:             node-role.kubernetes.io/master:NoSchedule # TAINTS
Unschedulable:      false
Lease:
  HolderIdentity:  master
  AcquireTime:     <unset>
  RenewTime:       Sun, 17 Jan 2021 22:13:43 +0000
# OUTPUT OMITTED
```

Allow the master server to run non-infrastructure pods (not recommended in production environment). The master node begins tainted for security and performance reasons.

```bash
student@master:~$ kubectl describe node | grep -i taint
Taints:             node-role.kubernetes.io/master:NoSchedule # Master
Taints:             <none> # Worker
```

Note the **minus sign (-)** at the end, which is the syntax to remove a taint.
> _The worker node does not have a taint so a "not found" error will raise_

```bash
student@master:~$ kubectl taint nodes --all node-role.kubernetes.io/master-
node/master untainted
error: taint "node-role.kubernetes.io/master" not found
```

---

Test all pods are running.
> _If not, you **may** find that the node has a new taint. Check the troubleshooting at the end of this page_

```bash
student@master:~$ kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-7dbc97f587-z579v   1/1     Running   3          22h
kube-system   calico-node-4z6hn                          1/1     Running   3          22h
kube-system   calico-node-dnwz9                          1/1     Running   1          7h5m
kube-system   coredns-66bff467f8-b4vdq                   1/1     Running   3          22h
kube-system   coredns-66bff467f8-vhmg2                   1/1     Running   3          22h
kube-system   etcd-master                                1/1     Running   3          22h
kube-system   kube-apiserver-master                      1/1     Running   3          22h
kube-system   kube-controller-manager-master             1/1     Running   3          22h
kube-system   kube-proxy-v8hwr                           1/1     Running   3          22h
kube-system   kube-proxy-zrcjm                           1/1     Running   1          7h5m
kube-system   kube-scheduler-master                      1/1     Running   3          22h
```

---

## Troubleshooting (tainted node, stuck pods)

After untainting the master node, you may find there is a new taint (```node.kubernetes.io/not-ready:NoSchedule```).

```bash
student@master:~$ kubectl describe node | grep -i taint
Taints:             node.kubernetes.io/not-ready:NoSchedule
Taints:             <none>
```

So, remove the taint.

```bash
student@master: ̃$ kubectl taint nodes --all node.kubernetes.io/not-ready-
node/master untainted
error: taint "node-role.kubernetes.io/not-ready" not found
```

Now, check if the dns and calico pods are in _Running_ state. It may take a while to transition from _Pending_.

**If** the coredns pods are stuck in ContainerCreating status, you may have to delete them.

```bash
student@master:~$ kubectl -n kube-system delete pod $(kubectl -n kube-system get pods -l "k8s-app=kube-dns")
```

When it finished, a new tunnel (_tunl0_) interface is available (```ip a``` command).

---

## References

* [Part 4: Deploy a simple app on the new cluster](../../week02/day8)
