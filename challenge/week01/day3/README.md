# Day 3 of #66DaysOfK8s

_Last update: 2021-01-13_

---

Today, I've learned to use a ServiceAccount which allows a Pod to communicate with the API server of the Kubernetes cluster.

> _Based on: [https://medium.com/better-programming/k8s-tips-using-a-serviceaccount-801c433d0023](https://medium.com/better-programming/k8s-tips-using-a-serviceaccount-801c433d0023)_

#kubernetes #learning #K8s #66DaysChallenge

---

## TL;DR

[Demo](#demo)

---

## Versions used

* macOS Catalina 10.15.7
* minikube version: v1.13.0
* kubectl Client Version: v1.17.4
* kubectl Server Version: v1.18.9

---

## Setup

* All tests runs on minikube.
* All pods are deployed on default namespace

---

## Using the namespace default ServiceAccount

Each namespace has a default ServiceAccount:

```bash
$ kubectl get sa -A| grep default
default           default      1    12h
istio-system      default      1    12h
kube-node-lease   default      1    12h
kube-public       default      1    12h
kube-system       default      1    12h
```

In the default ServiceAccount of the default namespace, there is a secret:

```bash
$ kubectl -n default get sa default -o yaml
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2021-01-12T21:40:27Z"
  name: default
  namespace: default
  resourceVersion: "320"
  selfLink: /api/v1/namespaces/default/serviceaccounts/default
  uid: 25d235a1-3799-495c-82f9-603e630e8078
secrets:
- name: default-token-tbmbs
```

```bash
$ kubectl -n default get secret $(kubectl -n default get sa default -o jsonpath='{.secrets[0].name}') -o yaml
# same as: kubectl -n default get secret default-token-tbmbs -o yaml
```

```yaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRU...0tCg== # (base64 cluster certificate, shortened)
  namespace: ZGVmYXVsdA== # (base64 namespace)
  token: ZXlKaGJHY2...UVpfdUhR # (base64 JWT for API server auth, shortened)
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: default
    kubernetes.io/service-account.uid: 25d235a1-3799-495c-82f9-603e630e8078
  creationTimestamp: "2021-01-12T21:40:27Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
      # ...
      f:type: {}
    manager: kube-controller-manager
    operation: Update
    time: "2021-01-12T21:40:27Z"
  name: default-token-tbmbs
  namespace: default
  resourceVersion: "317"
  selfLink: /api/v1/namespaces/default/secrets/default-token-tbmbs
  uid: 7c0156ac-0a99-45ca-94b3-18320285d317
type: kubernetes.io/service-account-token
```

Payload of the secret's JWT Token:

```bash
$ export SECRET_JWT_TOKEN=$(kubectl -n default get secret $(kubectl -n default get sa default -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 -d)
```

You can decode the JWT payload in websites like [jwt.io](https://jwt.io/), or by using jq (in Mac):

```bash
$ export SECRET_JWT_PAYLOAD=$(echo $SECRET_JWT_TOKEN | cut -d '.' -f2 | base64 -d)
```

```json
{
  "iss": "kubernetes/serviceaccount",
  "kubernetes.io/serviceaccount/namespace": "default",
  "kubernetes.io/serviceaccount/secret.name": "default-token-tbmbs",
  "kubernetes.io/serviceaccount/service-account.name": "default",
  "kubernetes.io/serviceaccount/service-account.uid": "25d235a1-3799-495c-82f9-603e630e8078",
  "sub": "system:serviceaccount:default:default"
}
```

---

Let's create a pod (```pod-default.yaml```)to test with.

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-default
spec:
 containers:
 - name: alpine
   image: alpine:3.9
   command:
   - "sleep"
   - "10000"
```

Create the pod:

```bash
$ kubectl apply -f pod-default.yaml
pod/pod-default created
```

No ```serviceAccountName``` key was specified in the last yaml, so the default ServiceAccount of Pod's namespace is used.

```bash
# The serviceAccountName key is set with the name of the default ServiceAccount
$ kubectl -n default get pod/pod-default -o jsonpath='{.spec.serviceAccountName}'
default
```

```bash
# The information of the ServiceAccount is mounted inside the container of the Pod, through the usage of a volume
$ kubectl -n default get pod/pod-default -o jsonpath='{.spec.containers[0].volumeMounts[0].mountPath}'
/var/run/secrets/kubernetes.io/serviceaccount
```

---

## Anonymous call of the API server

Let's install curl in the running pod:

```bash
$ kubectl -n default exec -ti pod-default -- apk add --update curl
```

Let's try to invoke the API server:
```bash
$ kubectl -n default exec -ti pod-default -- curl https://kubernetes/api/v1
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
command terminated with exit code 60
```

```bash
# An unauthenticated user is not allowed to
$ kubectl -n default exec -ti pod-default -- curl https://kubernetes/api/v1 --insecure
```

```json
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/api/v1\"",
  "reason": "Forbidden",
  "details": {

  },
  "code": 403
}
```

## Call using the ServiceAccount token

```bash
$ export POD_SA_TOKEN=$(kubectl -n default exec -ti pod-default -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```

```bash
$ kubectl -n default exec -ti pod-default -- curl -H "Authorization: Bearer $POD_SA_TOKEN" https://kubernetes/api/v1 --insecure
```

> _Response has been shortened for clarity_

```json
{
  "kind": "APIResourceList",
  "groupVersion": "v1",
  "resources": [
    {
      "name": "bindings",
      "singularName": "",
      "namespaced": true,
      "kind": "Binding",
      "verbs": [
        "create"
      ]
    },
. . .
    {
      "name": "services/status",
      "singularName": "",
      "namespaced": true,
      "kind": "Service",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    }
  ]
}
```

Now, let's try to list all the Pods within the default namespace with the same ServiceAccount:

```bash
$ kubectl -n default exec -ti pod-default -- curl -H "Authorization: Bearer $POD_SA_TOKEN" https://kubernetes/api/v1/namespaces/default/pods --insecure
```

```json
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```

---

## Using a custom ServiceAccount

### Creating a custom ServiceAccount

Let's create a new ServiceAccount in the default namespace, in ```demo-service-account.yaml```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: demo-sa
```

```bash
$ kubectl apply -f demo-service-account.yaml
serviceaccount/demo-sa created
```

### Creating a Role

A ServiceAccount has rights bound to it. Rights are known as Role or ClusterRole in Kubernetes. They are associated with a ServiceAccount, with RoleBinding and ClusterRoleBinding respectively.

Let's create a Role (```custom-role.yaml```)allowing to list all the Pods in the default namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: list-pods
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["list"]
```

```bash
$ kubectl apply -f custom-role.yaml
role.rbac.authorization.k8s.io/list-pods created
```

### Binding the Role with the ServiceAccount

Now, with ```custom-role-binding.yaml```, let's bind the Role with the ServiceAccount

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: list-pods_demo-sa
  namespace: default
subjects:
- kind: ServiceAccount
  name: demo-sa
  namespace: default
  #apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: list-pods
  apiGroup: rbac.authorization.k8s.io
```

---

## Using the ServiceAccount within a Pod

Let's create a pod (```pod-demo-sa.yaml```) with our last ServiceAccount:

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-demo-sa
spec:
 serviceAccountName: demo-sa
 containers:
 - name: alpine
   image: alpine:3.9
   command:
   - "sleep"
   - "10000"
```

```bash
$ kubectl apply -f pod-demo-sa.yaml
pod/pod-demo-sa created
```

Let's test API server with this new Pod.

```bash
kubectl -n default exec -ti pod-demo-sa -- apk add --update curl
export POD_DEMO_SA_TOKEN=$(kubectl -n default exec -ti pod-demo-sa -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```

The next call will work:
```bash
$ kubectl -n default exec -ti pod-demo-sa -- curl -H "Authorization: Bearer $POD_DEMO_SA_TOKEN" https://kubernetes/api/v1/namespaces/default/pods --insecure
```

```json
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces/default/pods",
    "resourceVersion": "22319"
  },
  "items": [
    {
      "metadata": {
        "name": "pod-default",
        "namespace": "default",
        "selfLink": "/api/v1/namespaces/default/pods/pod-default",
. . .
            "lastState": {

            },
            "ready": true,
            "restartCount": 0,
            "image": "alpine:3.9",
            "imageID": "docker-pullable://alpine@sha256:414e0518bb9228d35e4cd5165567fb91d26c6a214e9c95899e1e056fcd349011",
            "containerID": "docker://8c1d273d817a797e6ddaccc24458e6a6a09bd67a3dc0a306efda48d2dec01cfd",
            "started": true
          }
        ],
        "qosClass": "BestEffort"
      }
    }
  ]
}
```

---

# Demo

