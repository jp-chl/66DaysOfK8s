# Day 50 of #66DaysOfK8s

_Last update: 2021-03-01_

---
Today, in RBAC part 4, I have worked with ClusterRole and ClusterRoleBinding.

#kubernetes #learning #K8s #66DaysChallenge

---

## Setup

* Minikube, by default, gives you admin access to all resources. 
* Set an alias for kubectl (```alias k=kubectl```).
* Create a new user, for example, as in the [part 2](../day48).

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Tasks

* Understand differences between Roles and ClusterRoles, RoleBinding and ClusterRoleBinding
* Create a CR, a CRB, and test access to pods in different namespaces

---

## Understand differences between Roles and ClusterRoles, RoleBinding and ClusterRoleBinding

The difference between Role and ClusterRole is that the latter is cluster-wide and is not bound to a specific namespace. It gives permissions to all resources in every namespace.

A typical ClusterRole looks like a Role but its kind is different.

```yaml
# cr.yaml (ClusterRole example)
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sample-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Likewise, ClusteRoleBinding is similar to RoleBinding:

```yaml
# crb.yaml (ClusteRoleBinding example)
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sample-clusterrolebinding
subjects:
- kind: User
  name: myuser
  apiGroup: rbac.authorization.k8s.io
roleRef: # After creation, it is not editable (it needs to be recreated)
  kind: ClusterRole
  name: sample-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

---

## Create a CR, a CRB, and test access to pods in different namespaces

Create a new user, called "myuser". A [utility shell](../../week07/day49/newuser.sh) can be used. 

```bash
$ ../../week07/day49/newuser.sh myuser
Generating RSA private key, 2048 bit long modulus
........+++
.........................................+++
e is 65537 (0x10001)
Signature ok
subject=/CN=myuser/O=group1
Getting CA Private Key
User "myuser" set.
Context "myuser-context" created.
#
# minikube user
No resources found in default namespace.
# new user
Error from server (Forbidden): pods is forbidden: User "myuser" cannot list resource "pods" in API group "" in the namespace "default"
```

---

Test access in default namespace:

```bash
$ k get pods
No resources found in default namespace.
```

```bash
$ k get pods --user=myuser
No resources found in default namespace.
```

---

Let's create pods in different namespaces

```bash
$ k create ns ns1
namespace/ns1 created
```

```bash
$ k create ns ns2
namespace/ns2 created
```

```bash
$ k run nginx --image=nginx
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created
```

```bash
$ k run nginx --image=nginx -n ns1
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created
```

```bash
$ k run nginx --image=nginx -n ns2
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created
```

---

Test access:

```bash
$ k get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
default       nginx-598b589c46-szkrj             1/1     Running   0          46s
kube-system   coredns-f9fd979d6-cw9rm            1/1     Running   0          6m11s
kube-system   etcd-minikube                      1/1     Running   0          6m16s
kube-system   kube-apiserver-minikube            1/1     Running   0          6m16s
kube-system   kube-controller-manager-minikube   1/1     Running   0          6m16s
kube-system   kube-proxy-ztxtr                   1/1     Running   0          6m11s
kube-system   kube-scheduler-minikube            1/1     Running   0          6m16s
kube-system   storage-provisioner                1/1     Running   1          6m16s
ns1           nginx-598b589c46-nsc8z             1/1     Running   0          34s
ns2           nginx-598b589c46-9xvbt             1/1     Running   0          33s
```

Now, access to the user ```myuser``` must be cluster-wided for listing pods.

```bash
$ k get pods -A --user=myuser
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
default       nginx-598b589c46-szkrj             1/1     Running   0          46s
kube-system   coredns-f9fd979d6-cw9rm            1/1     Running   0          6m11s
kube-system   etcd-minikube                      1/1     Running   0          6m16s
kube-system   kube-apiserver-minikube            1/1     Running   0          6m16s
kube-system   kube-controller-manager-minikube   1/1     Running   0          6m16s
kube-system   kube-proxy-ztxtr                   1/1     Running   0          6m11s
kube-system   kube-scheduler-minikube            1/1     Running   0          6m16s
kube-system   storage-provisioner                1/1     Running   1          6m16s
ns1           nginx-598b589c46-nsc8z             1/1     Running   0          34s
ns2           nginx-598b589c46-9xvbt             1/1     Running   0          33s
```

---

If we try to access other resources such as a deployment, ```myuser``` won't be allowed to.

```bash
$ k get deploy -A
NAMESPACE     NAME      READY   UP-TO-DATE   AVAILABLE   AGE
default       nginx     1/1     1            1           68s
kube-system   coredns   1/1     1            1           6m39s
ns1           nginx     1/1     1            1           56s
ns2           nginx     1/1     1            1           55s
```

```bash
$ k get deploy -A --user=myuser                               ✔  at minikube ⎈  at 08:12:59 
Error from server (Forbidden): deployments.apps is forbidden: User "myuser" cannot list resource "deployments" in API group "apps" at the cluster scope
```

---

### Cleanup

```bash
$ k delete deploy nginx
deployment.apps "nginx" deleted
```

```bash
$ k delete deploy nginx -n ns1
deployment.apps "nginx" deleted
```

```bash
$ k delete deploy nginx -n ns2
deployment.apps "nginx" deleted
```

```bash
$ k delete ns ns1
namespace "ns1" deleted
```

```bash
$ k delete ns ns2
namespace "ns2" deleted
```

```bash
$ k delete -f cr.yaml
clusterrole.rbac.authorization.k8s.io "sample-clusterrole" deleted
```

```bash
$ k delete -f crb.yaml
clusterrolebinding.rbac.authorization.k8s.io "sample-clusterrolebinding" deleted
```

---

## References

* [Kubernetes: part 5 — RBAC authorization with a Role and RoleBinding example](https://itnext.io/kubernetes-part-5-rbac-authorization-with-a-role-and-rolebinding-example-765718e94f5a)

* [Kubernetes RBAC 101: authorization](https://www.cncf.io/blog/2020/08/28/kubernetes-rbac-101-authorization/)

* [Using RBAC Authorization (official site)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
