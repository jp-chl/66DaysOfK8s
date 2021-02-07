# Day 27 of #66DaysOfK8s

_Last update: 2021-02-06_

---

Today, I have worked with Sealed Secrets to encrypt K8s secrets with ```kubeseal``` command.

> _Based on: [Video demo of Anton Putra about Sealed Secrets](https://www.youtube.com/watch?v=ShGHCpUMdOg)_

Sealed Secrets helps to encrypts K8s secrets, useful to safely keep K8s secret manifests in GitHub repositories.

#kubernetes #learning #K8s #66DaysChallenge


---

## TL;DR


[Demo](#demo)

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0
* kubeseal: 0.13.1

---

## Setup

* All tests run on minikube.
* Anton Putra's github: [https://github.com/antonputra/tutorials/tree/main/044](https://github.com/antonputra/tutorials/tree/main/044)

---

## Tasks

* 

---

Create a file called ```01-kubeseal.yaml``` with the content available in this [link](https://raw.githubusercontent.com/antonputra/tutorials/main/044/k8s/01-kubeseal.yaml).

Namespace is explicitly defined as: kube-system. It can be any ns.

After apply this manifest the following objects (K8s kind) will be created: ```SealedSecret``` (CRD), ```ClusterRole```, ```ClusterRoleBinding```, ```Group```, ```Role```, ```RoleBinding```, ```ServiceAccount```, ```Deployment``` and a ```Service```.


```yaml
# 01-kubeseal.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations: {}
  labels:
    name: sealed-secrets-controller
  name: sealed-secrets-controller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
# Output omitted
```

Apply the last yaml.

```bash
$ kubectl apply -f yaml/01-kubeseal.yaml
serviceaccount/sealed-secrets-controller created
role.rbac.authorization.k8s.io/sealed-secrets-key-admin created
role.rbac.authorization.k8s.io/sealed-secrets-service-proxier created
rolebinding.rbac.authorization.k8s.io/sealed-secrets-controller created
rolebinding.rbac.authorization.k8s.io/sealed-secrets-service-proxier created
clusterrole.rbac.authorization.k8s.io/secrets-unsealer created
clusterrolebinding.rbac.authorization.k8s.io/sealed-secrets-controller created
service/sealed-secrets-controller created
deployment.apps/sealed-secrets-controller created
customresourcedefinition.apiextensions.k8s.io/sealedsecrets.bitnami.com created
```

---

A Sealed Secret controller has been generated and also a key pair. You can find them by checking its logs.

```bash
$ kubectl -n kube-system logs --tail=-1 -f -l name=sealed-secrets-controller
controller version: v0.13.1
2021/02/07 00:09:32 Starting sealed-secrets controller version: v0.13.1
2021/02/07 00:09:32 Searching for existing private keys
2021/02/07 00:09:33 New key written to kube-system/sealed-secrets-keyqp4db
2021/02/07 00:09:33 Certificate is
-----BEGIN CERTIFICATE-----
MIIErTCCApWgAwIBAgIQFZ0ZUoEJ6baX0VqOQ1CRwzANBgkqhkiG9w0BAQsFADAA
# Output omitted
-----END CERTIFICATE-----

2021/02/07 00:09:33 HTTP server serving on :8080
```

---

A .pem certificate can be fetched with ```kubeseal --fetch-cert``` command.
You can specify the namespace with the ```--controller-namespace \<namespace\>```. Default value: ```kube-system``` (```--controller-namespace kube-system```). 

```bash
$ kubeseal --controller-namespace kube-system --fetch-cert > cert.pem
```

```bash
$ cat ./cert.pem
-----BEGIN CERTIFICATE-----
MIIErTCCApWgAwIBAgIQFZ0ZUoEJ6baX0VqOQ1CRwzANBgkqhkiG9w0BAQsFADAA
# Output omitted
-----END CERTIFICATE-----
```

The certificate can be decode with ```openssl``` command.

```bash
$ openssl x509 -in cert.pem -text -noout|grep Validity -A2
        Validity
            Not Before: Feb  7 00:09:33 2021 GMT
            Not After : Feb  5 00:09:33 2031 GMT
```

---

In this example, the following simple K8s secret will be encrypted.

```yaml
# 03-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-encrypted-secret
  namespace: my-namespace # Specific namespace
type: Opaque
data:
  SECRET_WORD: TXlTZWNyZXRXb3Jk # Secret to encrypt
```

Do create the secret's namespace.

```bash
$ kubectl create ns my-namespace
namespace/my-namespace created
```

---

Let's encrypt the secret with ```kubeseal```.

First, generate the CRD SealedSecret manifest (```04-sealed-secret.yaml```).

```bash
$ kubeseal < yaml/03-secret.yaml --cert cert.pem --scope strict -o yaml > yaml/04-sealed-secret.yaml
```

```yaml
# 04-sealed-secret.yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret # CRD previously created
metadata:
  creationTimestamp: null
  name: my-encrypted-secret
  namespace: my-namespace
spec:
  encryptedData:
    SECRET_WORD: AgCHEgDKLO7XdS5E09oZPKdF0iFGq0QUfn5+975HLIlFTUp+6XK92Br36y1BsTsm7E8kMb+1veR1DBy1nhWwo # ... (omitted)
  template:
    metadata:
      creationTimestamp: null
      name: my-encrypted-secret
      namespace: my-namespace
    type: Opaque
```

Create the K8s secret (```my-encrypted-secret```). A "_my-encrypted-secret_" ```sealedsecret``` object is created as well.

```bash
$ kubectl apply -f yaml/04-sealed-secret.yaml
sealedsecret.bitnami.com/credentials created
```

```bash
$ kubectl -n my-namespace get sealedsecret,secret
NAME                                           AGE
sealedsecret.bitnami.com/my-encrypted-secret   3s

NAME                         TYPE                                  DATA   AGE
secret/default-token-m2hfs   kubernetes.io/service-account-token   3      7m25s
secret/my-encrypted-secret   Opaque                                1      3s
```

---

Now the secret can be decoded.

```bash
$ kubectl -n my-namespace get secret my-encrypted-secret -o jsonpath='{.data.SECRET_WORD}'|base64 -d
MySecretWord
```

---

## Cleanup

```bash
$ kubectl delete -f yaml/04-sealed-secret.yaml
sealedsecret.bitnami.com "my-encrypted-secret" deleted
```

```bash
$ kubectl delete ns my-namespace
namespace "my-namespace" delete
```

```bash
$ kubectl delete -f yaml/01-kubeseal.yaml
serviceaccount "sealed-secrets-controller" deleted
role.rbac.authorization.k8s.io "sealed-secrets-key-admin" deleted
role.rbac.authorization.k8s.io "sealed-secrets-service-proxier" deleted
rolebinding.rbac.authorization.k8s.io "sealed-secrets-controller" deleted
rolebinding.rbac.authorization.k8s.io "sealed-secrets-service-proxier" deleted
clusterrole.rbac.authorization.k8s.io "secrets-unsealer" deleted
clusterrolebinding.rbac.authorization.k8s.io "sealed-secrets-controller" deleted
service "sealed-secrets-controller" deleted
deployment.apps "sealed-secrets-controller" deleted
customresourcedefinition.apiextensions.k8s.io "sealedsecrets.bitnami.com" deleted
```

---


## References

* [Bitnami Sealed Secrets (official github)](https://github.com/bitnami-labs/sealed-secrets)

---

# Demo

[![asciicast](https://asciinema.org/a/WUZNy9cnj2RvX0giSkrY9UTzC.svg)](https://asciinema.org/a/WUZNy9cnj2RvX0giSkrY9UTzC)
