# Day 48 of #66DaysOfK8s

_Last update: 2021-02-28_

---
Today, in RBAC part 3, I have worked with Role and RoleBinding.

#kubernetes #learning #K8s #66DaysChallenge

---

## Setup

* Minikube, by default, gives you admin access to all resources. 
* Set an alias for kubectl (```alias k=kubectl```).
* Create a new user, for example, as in the [last part](../day48).

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Tasks

* Create a user and add it to kubeconfig and test access for list pods
* Create a Role, bind the new user and test its new access

---

## Create a user and add it to kubeconfig and test access for list pods


A convenient shell is available.

```bash
$ cat newuser.sh
openssl genrsa -out $1.key 2048
openssl req -new -key $1.key -out $1.csr -subj "/CN=$1/O=group1"
openssl x509 -req -in $1.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out $1.crt -days 500
kubectl config set-credentials $1 --client-certificate=$1.crt --client-key=$1.key
kubectl config set-context $1-context --cluster=minikube --user=$1
# 
#
kubectl get pods # list pods as minikube user (admin permissions)
kubectl get pods --user=$1 # (no permissions)
```

```bash
$ ./newuser.sh myuser
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

## Create a Role, bind the new user and test its new access

As described in [part 1](../day47/README.md), let's create a simple role for list all pods, get a specific one and watch their state changing (all in the default namespace).

apiVersion: rbac.authorization.k8s.io/v1
```yaml
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] 
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```bash
$ k apply -f role.yaml
role.rbac.authorization.k8s.io/pod-reader created
```

Now, let's bind that role to the new user.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: myuser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
$ k apply -f rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/read-pods created
```

Finally, you should be able to list pods in the default namespace.

```bash
$ k get pods --user=myuser
No resources found in default namespace.
```

---

### Cleanup

```bash
$ kubectl delete -f role.yaml
role.rbac.authorization.k8s.io "pod-reader" deleted
```

```bash
$ kubectl delete -f rolebinding.yaml
rolebinding.rbac.authorization.k8s.io "read-pods" deleted
```

---

## References

* [Kubernetes RBAC 101: authorization](https://www.cncf.io/blog/2020/08/28/kubernetes-rbac-101-authorization/)

* [Using RBAC Authorization (official site)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
