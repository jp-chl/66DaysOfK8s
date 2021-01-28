# Day 6 of #66DaysOfK8s

_Last update: 2021-01-16_

---

Today, I have worked in part 2 of a series of lessons in order to create a K8s cluster from scratch in GCP.
In this lesson I have configured the master node with K8s required software.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client

---

## Setup

* Master node created ([part 1 link](../day5/README.md))
* We'll be installing K8s version 1.18.1
* _Savings tip_: You can stop VM instances if you're not using them.
* If you, like me, have had issues connecting to the VM instances via SSH (after stop and start them regularly), there is a workaround ([check this link](../../workarounds.md)).

---

## Tasks

* Connect to master node
* Log in as root user. Update and upgrade the system
* Install a text editor (vi, nano, etc.)
* Install a container environment (docker or cri-o)
* Add a new repo for K8s
* Add a GPG key for the packages
* Update new repo
* Install main components (kubeadm, kubelet, kubectl)
* Configure a pod network
* Create a configuration file for the cluster
* Initialize the master
* Apply the network plugin configuration to your cluster
* Enable kubectl bash auto-completion
* Test installation

---

### Connect to master node

Using the SSK keys obtained in previous part, connect via ssh to the GCP VM instance associated with the master node:

```bash
# 1.2.3.4: replace with your own master node public ip
$ ssh -i "K8sPK1" student@1.2.3.4
The authenticity of host '1.2.3.4 (1.2.3.4)' can't be established.
ECDSA key fingerprint is SHA256:aknddjkj2ndbj213ndb23bndoi2ndbnjx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '1.2.3.4' (ECDSA) to the list of known hosts.

...

Last login: Fri Jan 15 11:46:26 2021 from 1.2.3.4
student@master:~$
```

---

### Log in as root user. Update and upgrade the system

```bash
$ student@master:~$ sudo -i
root@master:~#
```

```bash
# First and last command results for clarity
$ root@master:~$ apt-get update && apt-get upgrade -y
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease

...

0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

---

### Install a text editor (vi, nano, etc.)

```bash
# First and last command results for clarity
$ root@master:~$ apt-get install -y vim
Reading package lists... Done

...

0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

---

### Install a container environment (docker or cri-o)

Docker installation is way simpler, so let's continue with it.

```bash
# Only some command results are shown below
$ root@master:~$ apt-get install -y docker.io
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

---

### Add a new repo for K8s

Create a ```kubernetes.list``` file in ```/etc/apt/sources.list.d``` folder and add an entry for the main repo for K8s distribution.

```bash
$ root@master:~$ vim /etc/apt/sources.list.d/kubernetes.list
```

> _```kubernetes.list``` content_
```bash
deb http://apt.kubernetes.io/ kubernetes-xenial main
```

---

### Add a GPG key for the packages

Add a [GPG key](https://en.wikipedia.org/wiki/GNU_Privacy_Guard).

```bash
$ root@master:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
OK
```

---

### Update new repo

```bash
# First and last command results for clarity
$ root@master:~$ apt-get update
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease

...

Reading package lists... Done
```

---

### Install main components (kubeadm, kubelet, kubectl)

If you want to install the newest versions, you can omit the equal sign on the command line. Normally, newest versions might have bugs.

```bash
# Only some command results are shown
$ root@master:~$ apt-get install -y kubeadm=1.18.1-00 kubelet=1.18.1-00 kubectl=1.18.1-00
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

**Hold** the software at the recent but stable version

```bash
$ root@master:~$ apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
```

---

### Configure a pod network

We will use [Calico](https://www.projectcalico.org/) as a network plugin which allows [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

Once downloaded, loook for the expected IPv4 range for containers to use in the config. file.

The ```CALICO_IPV4POOL_CIDR``` must match the value given to kubeadm init in the following step, whatever the value may be

```bash
$ root@master:~$ more calico.yaml
```

> _```calico.yaml``` extract_
```yaml
    - name: FELIX_WIREGUARDMTU
        valueFrom:
        configMapKeyRef:
            name: calico-config
            key: veth_mtu
    # The default IPv4 pool to create on startup if none exists. Pod IPs will be
    # chosen from this range. Changing this value after installation will have
    # no effect. This should fall within `--cluster-cidr`.
    # - name: CALICO_IPV4POOL_CIDR
    #   value: "192.168.0.0/16"
    # Disable file logging so `kubectl logs` works.
    - name: CALICO_DISABLE_FILE_LOGGING
        value: "true"
```

Find the IP address of the primary interface of the master server.
> _In this case it is ```10.2.0.3```_

```bash
root@master:~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
    link/ether 42:01:0a:02:00:03 brd ff:ff:ff:ff:ff:ff
    inet 10.2.0.3/32 scope global dynamic ens4
       valid_lft 2236sec preferred_lft 2236sec
    inet6 fe80::4001:aff:fe02:3/64 scope link
       valid_lft forever preferred_lft forever
```

Add a local DNS alias for the master server.
Edit the ```/etc/hosts``` file and add the above IP address and assign the name k8smaster (node alias).

```bash
$ root@master:~$ vim /etc/hosts
```

> _```/etc/hosts``` content_
```bash
10.2.0.3 k8smaster # add this line
127.0.0.1 localhost

# ...
```

---

### Create a configuration file for the cluster

Create a configuration file for the cluster (```kubeadm-config.yaml```). For the moment, add only the control plane endpoint, K8s version and podSubnet values (```CALICO_IPV4POOL_CIDR``` in ```calico.yaml```).
Be sure to use the node alias (previous section), and not the IP.

```bash
$ root@master:~$ vim kubeadm-config.yaml
```

> _```kubeadm-config.yaml``` content_

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: 1.18.1       # Use the word "stable" for newest version
controlPlaneEndpoint: "k8smaster:6443" # Use the node alias not the IP
networking:
  podSubnet: 192.168.0.0/16 # Match de IP range from the Calico config file
```

---

### Initialize the master

```bash
$ root@master:~$ kubeadm init --config=kubeadm-config.yaml --upload-certs | tee kubeadm-init.out # Save output for future review

# Command results...
W0116 23:29:23.319523   20592 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.1
[preflight] Running pre-flight checks
	[WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local k8smaster] and IPs [10.96.0.1 10.2.0.3]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [master localhost] and IPs [10.2.0.3 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [master localhost] and IPs [10.2.0.3 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0116 23:29:44.519399   20592 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0116 23:29:44.520544   20592 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 17.502963 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
2cad6dbeb20d45d109ba9cdb1f2fcd3c63bbd9c0a9309b06ef7776fc7207717f
[mark-control-plane] Marking the node master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: d6ovfk.q9sqidty6s7jp8wu
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8smaster:6443 --token d6ovfk.q9sqidty6s7jp8wu \
    --discovery-token-ca-cert-hash sha256:be5a3ee9b1b2f710f76c389d48ef31753dfe0c2b8c36359c281fdcc2c7eee74b \
    --control-plane --certificate-key 2cad6dbeb20d45d109ba9cdb1f2fcd3c63bbd9c0a9309b06ef7776fc7207717f

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8smaster:6443 --token d6ovfk.q9sqidty6s7jp8wu \
    --discovery-token-ca-cert-hash sha256:be5a3ee9b1b2f710f76c389d48ef31753dfe0c2b8c36359c281fdcc2c7eee74b
```

---

Allow not-root admin level access to the cluster.

```bash
$ root@master:~$ exit
logout
student@master:~$
```

```bash
$ student@master:~$ mkdir -p $HOME/.kube
```

```bash
$ student@master:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```bash
$ student@master:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```bash
$ student@master:~$ cat .kube/config
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

### Apply the network plugin configuration to your cluster

Copy the calico.yaml file to the non-root user directory.

```bash
$ student@master:~$ sudo cp /root/calico.yaml .
```

Apply calico configuration.

```bash
$ student@master:~$ kubectl apply -f calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
poddisruptionbudget.policy/calico-kube-controllers created
```

---

### Enable kubectl bash auto-completion

```bash
$ student@master: ̃$ sudo apt-get install bash-completion -y
Reading package lists... Done
Building dependency tree
Reading state information... Done
bash-completion is already the newest version (1:2.8-1ubuntu1).
bash-completion set to manually installed.
The following package was automatically installed and is no longer required:
  libnuma1
Use 'sudo apt autoremove' to remove it.
0 upgraded, 0 newly installed, 0 to remove and 3 not upgraded.
```

```bash
$ student@master: ̃$ source <(kubectl completion bash)
```

```bash
$ student@master: ̃$ echo "source <(kubectl completion bash)" >> $HOME/.bashrc
```

Now, you can press tab while entering kubectl commands.

---

### Test installation


```bash
$ student@master:~$ kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.1", GitCommit:"7879fc12a63337efff607952a323df90cdc7a335", GitTreeState:"clean", BuildDate:"2020-04-08T17:38:50Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.1", GitCommit:"7879fc12a63337efff607952a323df90cdc7a335", GitTreeState:"clean", BuildDate:"2020-04-08T17:30:47Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

```bash
$ student@master:~$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   20m   v1.18.1
```

```bash
$ student@master:~$ kubectl -n kube-system get pods
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-7dbc97f587-z579v   1/1     Running   0          4m52s
calico-node-4z6hn                          1/1     Running   0          4m52s
coredns-66bff467f8-b4vdq                   1/1     Running   0          23m
coredns-66bff467f8-vhmg2                   1/1     Running   0          23m
etcd-master                                1/1     Running   0          24m
kube-apiserver-master                      1/1     Running   0          24m
kube-controller-manager-master             1/1     Running   0          24m
kube-proxy-v8hwr                           1/1     Running   0          23m
kube-scheduler-master                      1/1     Running   0          24m
```

---

The following are other values that might be include in the kubeadm-config.yaml file while creating the cluster.

```bash
$ student@master:~$ sudo kubeadm config print init-defaults

```

```yaml
W0116 23:55:52.925727    2478 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.18.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

---

## References

* [Part 3: Add worker node to the cluster](../day7)
