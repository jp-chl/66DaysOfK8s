# Day 48 of #66DaysOfK8s

_Last update: 2021-02-27_

---
Today, I have added a user to minikube.

#kubernetes #learning #K8s #66DaysChallenge

---

## Setup

* Minikube, by default, gives you admin access to all resources. 
* Set an alias for kubectl (```alias k=kubectl```)

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Tasks

* Create a user and added to kubeconfig

---

## Create a user

```bash
$ openssl genrsa -out myuser.key 2048
Generating RSA private key, 2048 bit long modulus
..................................................................+++
.......................................................................................+++
e is 65537 (0x10001)
```

```bash
$ openssl req -new -key myuser.key -out myuser.csr -subj "/CN=user1/O=group1"
```

```bash
$ openssl x509 -req -in myuser.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out myuser.crt -days 500
Signature ok
subject=/CN=user1/O=group1
Getting CA Private Key
```

```bash
$ ls myuser.*
myuser.csr myuser.key myuser.crt
```

```bash
$ kubectl config set-credentials myuser --client-certificate=myuser.csr --client-key=myuser.key
User "myuser" set.
```

```bash
$ kubectl config set-context myuser-context --cluster=minikube --user=myuser
Context "myuser-context" created.
```

```bash
$ kubectl config view
# Output omitted
 - name: minikube
   user:
     client-certificate: ....../client.crt
     client-key: ....../client.key
 - name: myuser
   user:
     client-certificate: ....../kubernetes/66DaysOfK8s/challenge/week07/day48/cert/myuser.csr
     client-key: ....../kubernetes/66DaysOfK8s/challenge/week07/day48/cert/myuser.key
```

---

## References

* [Kubernetes RBAC 101: authorization](https://www.cncf.io/blog/2020/08/28/kubernetes-rbac-101-authorization/)

* [Using RBAC Authorization (official site)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)


