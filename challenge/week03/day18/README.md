# Day 18 of #66DaysOfK8s

_Last update: 2021-01-28_

---

Today, I have worked doing basic node maintenance of a cluster (created from scratch in GCP).
This second session involves upgrading Kubernetes cluster version, in the master node and the worker as well.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client

---

## Setup

* K8s cluster already created in GCP from scratch. Check the instructions in this [link](../../week01/day5/README.md).

---

## Tasks

* Upgrade the cluster Kubernetes version

---

### Connect to the master node

Using the SSK keys obtained in a [previous lab](../../week01/day5/README.md), connect via ssh to the GCP VM instance associated with the **master** node:

```bash
# 1.2.3.4: replace with your own master node public IP
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

### Upgrade master node

Update APT.

```bash
student@master:~$ sudo apt update
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease
# ...
Reading state information... Done
24 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

---

Currently, the cluster should be using K8s 1.18.1 version:

```bash
student@master:~$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   11d   v1.18.1
worker   Ready    <none>   11d   v1.18.1
```

We'll be upgrading to Kubernetes 1.19.0 version. Look for it in the following command output:

```bash
student@master:~$ sudo apt-cache madison kubeadm
   kubeadm |  1.20.2-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.1-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.0-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.7-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   # ...
   kubeadm |  1.19.0-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.15-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   # ...
   kubeadm | 1.17.17-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   # ...
   kubeadm |   1.5.7-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
```

---

Remove the [hold](../../week01/day6#install-main-components-kubeadm-kubelet-kubectl) on kubeadm:

```bash
student@master:~$ sudo apt-mark unhold kubeadm
Canceled hold on kubeadm.
```

Update the package:

```bash
student@master:~$ sudo apt-get install -y kubeadm=1.19.0-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libnuma1
Use 'sudo apt autoremove' to remove it.
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 23 not upgraded.
Need to get 7759 kB of archives.
After this operation, 705 kB disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.19.0-00 [7759 kB]
Fetched 7759 kB in 1s (14.9 MB/s)
(Reading database ... 65906 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.19.0-00_amd64.deb ...
Unpacking kubeadm (1.19.0-00) over (1.18.1-00) ...
Setting up kubeadm (1.19.0-00) ...
```

Hold the package again

```bash
student@master:~$ sudo apt-mark hold kubeadm
kubeadm set on hold.
```

```bash
student@master:~$ sudo kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.0", GitCommit:"e19964183377d0ec2052d1f1fa930c4d7575bd50", GitTreeState:"clean", BuildDate:"2020-08-26T14:28:32Z", GoVersion:"go1.15", Compiler:"gc", Platform:"linux/amd64"}
```

---

A master update requires to remove all pods but the daemonSets.

```bash
student@master:~$ kubectl drain master --ignore-daemonsets
node/master cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-4z6hn, kube-system/kube-proxy-v8hwr
evicting pod kube-system/calico-kube-controllers-7dbc97f587-z579v
evicting pod kube-system/coredns-66bff467f8-b4vdq
evicting pod kube-system/coredns-66bff467f8-vhmg2
pod/calico-kube-controllers-7dbc97f587-z579v evicted
pod/coredns-66bff467f8-b4vdq evicted
pod/coredns-66bff467f8-vhmg2 evicted
node/master evicted
```

---

Prepare the upgrade with ```kubeadm upgrade plan``` command.

```bash
student@master:~$ sudo kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.18.1
[upgrade/versions] kubeadm version: v1.19.0
I0128 22:21:52.626111   30763 version.go:252] remote version is much newer: v1.20.2; falling back to: stable-1.19
[upgrade/versions] Latest stable version: v1.19.7
[upgrade/versions] Latest stable version: v1.19.7
[upgrade/versions] Latest version in the v1.18 series: v1.18.15
[upgrade/versions] Latest version in the v1.18 series: v1.18.15

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
kubelet     2 x v1.18.1   v1.18.15

Upgrade to the latest version in the v1.18 series:

COMPONENT                 CURRENT   AVAILABLE
kube-apiserver            v1.18.1   v1.18.15
kube-controller-manager   v1.18.1   v1.18.15
kube-scheduler            v1.18.1   v1.18.15
kube-proxy                v1.18.1   v1.18.15
CoreDNS                   1.6.7     1.7.0
etcd                      3.4.3-0   3.4.3-0

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.18.15

_____________________________________________________________________

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
kubelet     2 x v1.18.1   v1.19.7

Upgrade to the latest stable version:

COMPONENT                 CURRENT   AVAILABLE
kube-apiserver            v1.18.1   v1.19.7
kube-controller-manager   v1.18.1   v1.19.7
kube-scheduler            v1.18.1   v1.19.7
kube-proxy                v1.18.1   v1.19.7
CoreDNS                   1.6.7     1.7.0
etcd                      3.4.3-0   3.4.9-1

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.19.7

Note: Before you can perform this upgrade, you have to update kubeadm to v1.19.7.

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________
```

---

The command (```kubeadm upgrade apply v1.19.0```) output should mention:

```bash
[upgrade/version] You have chosen to change the cluster version to "v1.19.0"
[upgrade/versions] Cluster version: v1.18.1
[upgrade/versions] kubeadm version: v1.19.0
```

**Do the upgrade**. Answer yes to proceed:

```bash
student@master:~$ sudo kubeadm upgrade apply v1.19.0
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.19.0"
[upgrade/versions] Cluster version: v1.18.1
[upgrade/versions] kubeadm version: v1.19.0
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.19.0"...
Static pod: kube-apiserver-master hash: 5d21045e3b9a6aeb7611eb6a3c2e042b
Static pod: kube-controller-manager-master hash: a2e7dbae641996802ce46175f4f5c5dc
Static pod: kube-scheduler-master hash: 363a5bee1d59c51a98e345162db75755
[upgrade/etcd] Upgrading to TLS for etcd
Static pod: etcd-master hash: de7e07a4a4ae3f4e585e9c8b1784dee4
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Renewing etcd-server certificate
[upgrade/staticpods] Renewing etcd-peer certificate
[upgrade/staticpods] Renewing etcd-healthcheck-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2021-01-28-22-25-39/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: etcd-master hash: de7e07a4a4ae3f4e585e9c8b1784dee4
Static pod: etcd-master hash: de7e07a4a4ae3f4e585e9c8b1784dee4
Static pod: etcd-master hash: ce96626d22fbac4bc404ee9ca4c106ad
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests466204301"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2021-01-28-22-25-39/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-apiserver-master hash: 5d21045e3b9a6aeb7611eb6a3c2e042b
Static pod: kube-apiserver-master hash: 5d21045e3b9a6aeb7611eb6a3c2e042b
Static pod: kube-apiserver-master hash: 2277181ee3fbfee31641c79565ecfcb2
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2021-01-28-22-25-39/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-controller-manager-master hash: a2e7dbae641996802ce46175f4f5c5dc
Static pod: kube-controller-manager-master hash: ead3b8933eb874ce423dbc0be136df58
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2021-01-28-22-25-39/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-scheduler-master hash: 363a5bee1d59c51a98e345162db75755
Static pod: kube-scheduler-master hash: 23d2ea3ba1efa3e09e8932161a572387
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.19" in namespace kube-system with the configuration for the kubelets in the cluster
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.19.0". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

---

If we check nodes statuses, the master scheduling is now disabled and K8s version is still the old version. We must update all the required software and restart the daemons.

```bash
student@master:~$ kubectl get nodes
NAME     STATUS                     ROLES    AGE   VERSION
master   Ready,SchedulingDisabled   master   11d   v1.18.1
worker   Ready                      <none>   11d   v1.18.1
```

---

Remove [hold](../../week01/day6#install-main-components-kubeadm-kubelet-kubectl) on kubelet.

```bash
student@master:~$ sudo apt-mark unhold kubelet
Canceled hold on kubelet.
```

Remove hold on kubectl.

```bash
student@master:~$ sudo apt-mark unhold kubectl
Canceled hold on kubectl.
```

Upgrade kubelet.

```bash
student@master:~$ sudo apt-get install -y kubelet=1.19.0-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libnuma1
Use 'sudo apt autoremove' to remove it.
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 23 not upgraded.
Need to get 18.2 MB of archives.
After this operation, 3297 kB disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.19.0-00 [18.2 MB]
Fetched 18.2 MB in 2s (11.9 MB/s)
(Reading database ... 65906 files and directories currently installed.)
Preparing to unpack .../kubelet_1.19.0-00_amd64.deb ...
Unpacking kubelet (1.19.0-00) over (1.18.1-00) ...
Setting up kubelet (1.19.0-00) ...
```

Upgrade kubectl

```bash
student@master:~$ sudo apt-get install -y kubectl=1.19.0-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libnuma1
Use 'sudo apt autoremove' to remove it.
The following packages will be upgraded:
  kubectl
1 upgraded, 0 newly installed, 0 to remove and 23 not upgraded.
Need to get 8349 kB of archives.
After this operation, 1024 kB disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.19.0-00 [8349 kB]
Fetched 8349 kB in 1s (11.3 MB/s)
(Reading database ... 65906 files and directories currently installed.)
Preparing to unpack .../kubectl_1.19.0-00_amd64.deb ...
Unpacking kubectl (1.19.0-00) over (1.18.1-00) ...
Setting up kubectl (1.19.0-00) ...
```

Hold updates for kubelet.

```bash
student@master:~$ sudo apt-mark hold kubelet
kubelet set on hold.
```

And hold for kubectl.

```bash
student@master:~$ sudo apt-mark hold kubectl
kubectl set on hold.
```

---

Restart daemons.

```bash
student@master:~$ sudo systemctl daemon-reload
# (no output expected)
```

```bash
student@master:~$ sudo systemctl restart kubelet
# (no output expected)
```

Check updated version on master node.

```bash
student@master:~$ kubectl get nodes
NAME     STATUS                     ROLES    AGE   VERSION
master   Ready,SchedulingDisabled   master   11d   v1.19.0
worker   Ready                      <none>   11d   v1.18.1
```

> _Updating other master nodes requires the same procedure, except the command ```sudo kubeadm updgrade node``` must be applied instead of ```sudo kubeadm update apply```._

---

Enable scheduling for the master node.

```bash
student@master:~$ kubectl uncordon master
node/master uncordoned
```

Notice ```Ready```status now.

```bash
student@master:~$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   11d   v1.19.0
worker   Ready    <none>   11d   v1.18.1
```

---

### Upgrade worker node

Using the SSK keys obtained in a [previous lab](../../week01/day5/README.md), connect via ssh to the GCP VM instance associated with the worker node:

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

---

Remove hold on kubeadm too.

```bash
student@worker:~$ sudo apt-mark unhold kubeadm
Canceled hold on kubeadm.
```

---

Upgrade kubeadm also to 1.19.0-00 version.

```bash
student@worker:~$ sudo apt-get update
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease
# ...
Reading package lists... Done
```

```bash
student@worker:~$ sudo apt-get install -y kubeadm=1.19.0-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libnuma1
Use 'sudo apt autoremove' to remove it.
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 23 not upgraded.
Need to get 7759 kB of archives.
After this operation, 705 kB disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.19.0-00 [7759 kB]
Fetched 7759 kB in 1s (12.8 MB/s)
(Reading database ... 65906 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.19.0-00_amd64.deb ...
Unpacking kubeadm (1.19.0-00) over (1.18.1-00) ...
Setting up kubeadm (1.19.0-00) ...
```

And hold kubeadm again.

```bash
student@worker:~$ sudo apt-mark hold kubeadm
kubeadm set on hold.
```

---

Just like in the master node, drain pods in the worker node. **Do this in the master node**.

```bash
student@master:~$ kubectl drain worker --ignore-daemonsets
node/worker cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-dnwz9, kube-system/kube-proxy-2l8pw
evicting pod kube-system/calico-kube-controllers-7dbc97f587-5h46c
evicting pod kube-system/coredns-f9fd979d6-d48bv
evicting pod kube-system/coredns-f9fd979d6-klk6t
pod/calico-kube-controllers-7dbc97f587-5h46c evicted
pod/coredns-f9fd979d6-klk6t evicted
pod/coredns-f9fd979d6-d48bv evicted
node/worker evicted
```

---

Back to the worker node, upgrade it.

```bash
student@worker:~$ sudo kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
```

---

Remove hold on kubelet.

```bash
student@worker:~$ sudo apt-mark unhold kubelet
Canceled hold on kubelet.
```

Remove hold on kubectl.

```bash
student@worker:~$ sudo apt-mark unhold kubectl
Canceled hold on kubectl.
```

Upgrade kubelet.

```bash
student@worker:~$ sudo apt-get install -y kubelet=1.19.0-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libnuma1
Use 'sudo apt autoremove' to remove it.
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 23 not upgraded.
Need to get 18.2 MB of archives.
After this operation, 3297 kB disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.19.0-00 [18.2 MB]
Fetched 18.2 MB in 1s (16.6 MB/s)
(Reading database ... 65906 files and directories currently installed.)
Preparing to unpack .../kubelet_1.19.0-00_amd64.deb ...
Unpacking kubelet (1.19.0-00) over (1.18.1-00) ...
Setting up kubelet (1.19.0-00) ...
```

Upgrade kubectl

```bash
student@worker:~$ sudo apt-get install -y kubectl=1.19.0-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libnuma1
Use 'sudo apt autoremove' to remove it.
The following packages will be upgraded:
  kubectl
1 upgraded, 0 newly installed, 0 to remove and 23 not upgraded.
Need to get 8349 kB of archives.
After this operation, 1024 kB disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.19.0-00 [8349 kB]
Fetched 8349 kB in 1s (16.5 MB/s)
(Reading database ... 65906 files and directories currently installed.)
Preparing to unpack .../kubectl_1.19.0-00_amd64.deb ...
Unpacking kubectl (1.19.0-00) over (1.18.1-00) ...
Setting up kubectl (1.19.0-00) ...
```

```bash
student@worker:~$ kubectl get nodes
NAME     STATUS                     ROLES    AGE   VERSION
master   Ready                      master   11d   v1.19.0
worker   Ready,SchedulingDisabled   <none>   11d   v1.19.0
```

Hold updates for kubelet.

```bash
student@worker:~$ sudo apt-mark hold kubelet
kubelet set on hold.
```

And hold for kubectl.

```bash
student@worker:~$ sudo apt-mark hold kubectl
kubectl set on hold.
```

---

Restart daemons.

```bash
student@worker:~$ sudo systemctl daemon-reload
# (no output expected)
```

```bash
student@worker:~$ sudo systemctl restart kubelet
# (no output expected)
```

---

**Back to the master node**. Enable scheduling for the worker node.

```bash
student@master:~$ kubectl uncordon worker
node/worker uncordoned
```

```bash
student@master:~$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   11d   v1.19.0
worker   Ready    <none>   11d   v1.19.0
```

---
## References

* [Upgrade K8s cluster (official site)](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
