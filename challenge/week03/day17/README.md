# Day 17 of #66DaysOfK8s

_Last update: 2021-01-27_

---

Today, I have started two lessons in order to do basic node maintenance of a cluster (created from scratch in GCP).
This first part involves backing up the etcd database.

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

* Backup the etcd database
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

### Back up etcd database

There are many alternatives to do this, recommended, task. In this case, snapshot command is used.

Look for the data directory of the etcd daemon process. All of its setting is in the manifest (in this case ```/var/lib/etcd```).

```bash
student@master:~$ sudo grep data-dir /etc/kubernetes/manifests/etcd.yaml
    - --data-dir=/var/lib/etcd
```

In the kube-system namespace, there is one process for etcd (```etcd-master````).

```bash
student@master:~$ kubectl -n kube-system get pods
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-7dbc97f587-z579v   1/1     Running   8          11d
calico-node-4z6hn                          1/1     Running   8          11d
calico-node-dnwz9                          1/1     Running   4          10d
coredns-66bff467f8-b4vdq                   1/1     Running   8          11d
coredns-66bff467f8-vhmg2                   1/1     Running   8          11d
etcd-master                                1/1     Running   8          11d
kube-apiserver-master                      1/1     Running   8          11d
kube-controller-manager-master             1/1     Running   8          11d
kube-proxy-v8hwr                           1/1     Running   8          11d
kube-proxy-zrcjm                           1/1     Running   4          10d
kube-scheduler-master                      1/1     Running   8          11d
```

Enter interactive mode to that process. To manage etcd you can use the ```etcdctl``` command. Find 3 files that etcdctl command requires (```ca.crt```, ```server.crt``` and ```server.key```) located in ```/etc/kubernetes/pki/etcd``` folder.

```bash
student@master:~$ kubectl -n kube-system exec -it etcd-master -- sh
# 
# cd /etc/kubernetes/pki/etcd
# 
# ls -l
total 32
-rw-r--r-- 1 root root 1017 Jan 16 23:29 ca.crt
-rw------- 1 root root 1675 Jan 16 23:29 ca.key
-rw-r--r-- 1 root root 1094 Jan 16 23:29 healthcheck-client.crt
-rw------- 1 root root 1679 Jan 16 23:29 healthcheck-client.key
-rw-r--r-- 1 root root 1127 Jan 16 23:29 peer.crt
-rw------- 1 root root 1679 Jan 16 23:29 peer.key
-rw-r--r-- 1 root root 1127 Jan 16 23:29 server.crt
-rw------- 1 root root 1675 Jan 16 23:29 server.key
```

You can pass those 3 files as parameters to etcdctl or set environment variables.

> _We're applying etcdctl commands within etcd-master Pod, and then from the master (via kubectl)._

---

Check database health:

```bash
# etcdctl endpoint health
{"level":"warn","ts":"2021-01-28T00:49:33.553Z","caller":"clientv3/retry_interceptor.go:61","msg":"retrying of unary invoker failed","target":"endpoint://client-ceb95ffa-b032-4cf3-a134-c98ea6b87bec/127.0.0.1:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest connection error: connection closed"}
127.0.0.1:2379 is unhealthy: failed to commit proposal: context deadline exceeded
Error: unhealthy cluster
```

> _An error is expected because we haven't passed the peer cert and key, and the certificate authority as environment variables_

```bash
# export ETCDCTL_API=3
# export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
# export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
# export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
# 
# env|grep ETC
ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
ETCDCTL_API=3
ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
```

```bash
# etcdctl endpoint health
127.0.0.1:2379 is healthy: successfully committed proposal: took = 71.873859ms
```

---

Database instances in the cluster:

```bash
# etcdctl member list
9016b8822faae9ee, started, master, https://10.2.0.3:2380, https://10.2.0.3:2379, false
```

> _Last output is described with ```etcdctl member list --help``` (ID, Status, Name, Peer Addrs, Client Addrs, Is Learner)_

---

Cluster status in table format:

```bash
# etcdctl --endpoints=https://127.0.0.1:2379 -w table endpoint status --cluster
+-----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|       ENDPOINT        |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+-----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://10.2.0.3:2379 | 9016b8822faae9ee |   3.4.3 |  5.0 MB |      true |      false |        10 |      77109 |              77109 |        |
+-----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
```

---

Back up the database(s):

We are going to save the snapshot in the master node, so we're executing the etcdctl command in it (by applying _"kubectl ... -- \<pod command>"_) and passing the environment variables as parameters.

```bash
student@master:~$ kubectl -n kube-system exec -it etcd-master -- sh -c "ETCDCTL_API=3 ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key etcdctl endpoint health"
127.0.0.1:2379 is healthy: successfully committed proposal: took = 12.734277ms
```

```bash
student@master:~$ kubectl -n kube-system exec -it etcd-master -- sh -c "ETCDCTL_API=3 ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key etcdctl --endpoints=https://127.0.0.1:2379 member list"
9016b8822faae9ee, started, master, https://10.2.0.3:2380, https://10.2.0.3:2379, false
```

Backing up in the ```/var/lib/etcd``` directory.

```bash
student@master:~$ sudo ls -l /var/lib/etcd/
total 4
drwx------ 4 root root 4096 Jan 28 00:32 member
```

```bash
student@master:~$ kubectl -n kube-system exec -it etcd-master -- sh -c "ETCDCTL_API=3 ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key etcdctl --endpoints=https://127.0.0.1:2379 snapshot save /var/lib/etcd/snapshot.db"
{"level":"info","ts":1611796898.6340106,"caller":"snapshot/v3_snapshot.go:110","msg":"created temporary db file","path":"/var/lib/etcd/snapshot.db.part"}
{"level":"warn","ts":"2021-01-28T01:21:38.644Z","caller":"clientv3/retry_interceptor.go:116","msg":"retry stream intercept"}
{"level":"info","ts":1611796898.6444745,"caller":"snapshot/v3_snapshot.go:121","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
{"level":"info","ts":1611796898.7759843,"caller":"snapshot/v3_snapshot.go:134","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","took":0.141739585}
{"level":"info","ts":1611796898.7761414,"caller":"snapshot/v3_snapshot.go:143","msg":"saved","path":"/var/lib/etcd/snapshot.db"}
```

```bash
student@master:~$ sudo ls -l /var/lib/etcd/
total 4908
drwx------ 4 root root    4096 Jan 28 00:32 member
-rw------- 1 root root 5017632 Jan 28 01:21 snapshot.db
```

```bash
student@master:~$ mkdir $HOME/backup
student@master:~$ sudo cp /var/lib/etcd/snapshot.db $HOME/backup/snapshot.db-$(date +%m-%d-%y)
student@master:~$ sudo cp /root/kubeadm-config.yaml $HOME/backup/kubeadm-config.yaml-$(date +%m-%d-%y)
student@master:~$ sudo cp -r /etc/kubernetes/pki/etcd $HOME/backup
student@master:~$ mv $HOME/backup/etcd $HOME/backup/etcd-$(date +%m-%d-%y)
student@master:~$ ls -l $HOME/backup
total 4912
drwxr-xr-x 2 root root    4096 Jan 28 01:28 etcd-01-28-21
-rw-r--r-- 1 root root     298 Jan 28 01:28 kubeadm-config.yaml-01-28-21
-rw------- 1 root root 5017632 Jan 28 01:28 snapshot.db-01-28-21
```

---

## References

* [Operating etcd clusters for Kubernetes (official site)](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
