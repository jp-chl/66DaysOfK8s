# Day 51 of #66DaysOfK8s

_Last update: 2021-03-02_

---
Today, in RBAC part 5, I have worked with Service accounts linked with Roles.

_(Based on [medium article by Luc Juggery](https://medium.com/better-programming/k8s-tips-using-a-serviceaccount-801c433d0023))_

#kubernetes #learning #K8s #66DaysChallenge

---

## Setup

* Minikube, by default, gives you admin access to all resources. 
* Set an alias for kubectl (```alias k=kubectl```) plus execute the following command: ```complete -F __start_kubectl k```.


---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Tasks

* Validate access to Kubernetes API with the default Service account
* Create a custom Service account and link it to a Role
* Test access with newly Service account

---

## Validate access with default Service account

Let's create a simple Pod with curl already installed.

```bash
$ k run --generator=run-pod/v1 tester --image=nectiadocker2000/podtesterspring:v2
pod/tester created
```

As a service account was not specified, the default one was injected automatically.

```bash
$ k get pod tester -o yaml | grep serviceAccountName
  serviceAccountName: default
```


Get the default secret's JWT Token:

```bash
$ echo $(k -n default get secret $(kubectl -n default get sa $(k get pod tester -o jsonpath='{.spec.serviceAccountName}') -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}') | base64 -d | cut -d '.' -f2 | base64 -d
```

```json
{"iss":"kubernetes/serviceaccount","kubernetes.io/serviceaccount/namespace":"default","kubernetes.io/serviceaccount/secret.name":"default-token-pfp5h","kubernetes.io/serviceaccount/service-account.name":"default","kubernetes.io/serviceaccount/service-account.uid":"837578dc-28a8-4dcd-8001-3a87ae717497","sub":"system:serviceaccount:default:default"}
```

---

Unless the JWT token is used, an attempt to call K8s API will fail.

```bash
$ k exec -ti tester -- curl -i https://kubernetes/api/v1
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
command terminated with exit code 60
```

```bash
k exec -ti tester -- curl -i https://kubernetes/api/v1 --insecure
HTTP/2 403
cache-control: no-cache, private
content-type: application/json
x-content-type-options: nosniff
content-length: 239
date: Wed, 03 Mar 2021 03:24:02 GMT

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

---

The Token is mounted in a Pod's volume.

```bash
$ k get pod tester -o jsonpath='{.spec.containers[0].volumeMounts[0].mountPath}'; echo
/var/run/secrets/kubernetes.io/serviceaccount
```

```bash
$ export TOKEN_MOUNTED_DIR=$(k get pod tester -o jsonpath='{.spec.containers[0].volumeMounts[0].mountPath}')
```

```bash
$ k exec -ti tester -- cat ${TOKEN_MOUNTED_DIR}/token
eyJhbGciOiJSUzI1NiIsImtpZCI6IjRiUm5mdGdWQXBtMjBPTVdkTTJaWHMxWWFQS2NNNmd5cDF6c0NtTDBWWVEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tcGZwNWgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjgzNzU3OGRjLTI4YTgtNGRjZC04MDAxLTNhODdhZTcxNzQ5NyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.QPxjlEqQ3dICkFot_ZroMNeLWe2aSQ2ZY1ZeR0DcNRmigJiC2WU1nb-9SfyKYrZiIcZgNQMaNdFpEL1YIqNiY4Be5DEY72sGqWkmLh1SdoIvR3iIF_lma4yeOttsqfA_VkR5G_VxjsUjtb5hTAT6W9mkc33SYmaTWEcHys0TT-uzVX_wGs00tpXAFnrUOifJsaEM1xtsc_fuV4PPhxQjsY26DrMUqRq2EwT4xcyUYXgxdL3UiD5hVk8UONpMJC9rkd5niDFBXhLjVNOddCqqjat6NL-JiT-mEIHeogUxkr2lmJBI8F_HwTJderdqxg_OCFI25G39Eoj8ncIxXYqXxw
```

```bash
$ export POD_SA_TOKEN=$(k exec -ti tester -- cat ${TOKEN_MOUNTED_DIR}/token)
```

Now, the API call will work.

```bash
$ k exec -ti tester -- curl -H "Authorization: Bearer $POD_SA_TOKEN" -i https://kubernetes/api/v1 --insecure
HTTP/2 200
cache-control: no-cache, private
content-type: application/json
date: Wed, 03 Mar 2021 03:31:04 GMT

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
# Output omitted
    {
      "name": "pods",
      "singularName": "",
      "namespaced": true,
      "kind": "Pod",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "po"
      ],
      "categories": [
        "all"
      ],
      "storageVersionHash": "xPOwRZ+Yhw8="
    },
# Output omitted
```

However, a simple Pod list request is forbidden (_https://kubernetes/api/v1/namespaces/default/pods_).

```bash
$ k exec -ti tester -- curl -H "Authorization: Bearer $POD_SA_TOKEN" -i https://kubernetes/api/v1/namespaces/default/pods --insecure
HTTP/2 403
cache-control: no-cache, private
content-type: application/json
x-content-type-options: nosniff
content-length: 331
date: Wed, 03 Mar 2021 03:33:52 GMT

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

## Create a custom Service account and link it to a Role

```bash
$ k create sa new-sa
serviceaccount/new-sa created
```

```bash
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: list-pods
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
EOF
```

```bash
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: list-pods_demo-sa
  namespace: default
subjects:
- kind: ServiceAccount
  name: new-sa
  namespace: default
  #apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: list-pods
  apiGroup: rbac.authorization.k8s.io
EOF
```

---

Let's create a new Pod and associate it with the last service account.

```bash
$ k run --generator=run-pod/v1 tester2 --image=nectiadocker2000/podtesterspring:v2 -o yaml --dry-run > pod2.yaml
```

```yaml
# pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: tester2
  name: tester2
spec:
  # add service account here ( serviceAccountName = "new-sa")
  containers:
  - image: nectiadocker2000/podtesterspring:v2
    name: tester2
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
$ k apply -f pod2.yaml
pod/tester2 created
```

```bash
$ k get pod tester2 -o yaml | grep serviceAccountName | grep -v "{"
  serviceAccountName: new-sa
```

---

## Test access with newly Service account

```bash
$ export TOKEN2_MOUNTED_DIR=$(k get pod tester2 -o jsonpath='{.spec.containers[0].volumeMounts[0].mountPath}')
```

```bash
$ k exec -ti tester2 -- cat ${TOKEN2_MOUNTED_DIR}/token       ✔  at minikube ⎈  at 00:49:32 
eyJhbGciOiJSUzI1NiIsImtpZCI6IjRiUm5mdGdWQXBtMjBPTVdkTTJaWHMxWWFQS2NNNmd5cDF6c0NtTDBWWVEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im5ldy1zYS10b2tlbi1wZDk3YiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuZXctc2EiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIxYTMxZjM4Zi00MWM5LTQ2MzctYmI1Zi02ZDRiZDJjNGM4MGQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpuZXctc2EifQ.a2SdNnW9BfydQblF6DwKxY5eww8Xwl8eFew111AMQalZpjAc13y_dcAxxe6k-iaYbuVQcYnMbZVkde7FG19kJUb4DtuTNoBfrmtIXEpdviTsR__R7fURWxt4qFefzdoAkZFVDtV6GnHMOCY9UIn6fJOTJDtysZRvv_L46du75jtzE3O5wVEqOkWFJDCxK12Qn8js488VNXfQ-1K1a-ubvyKJmSBjxQ7RlZRdYuyreOB3-0c2VGickPSSXnMCyTEvE2LYOVDpyvljSB1-APNJ91V2ZfpGXTVkPsEzrMukUSfcU3Lswqnza8SgMZTig4wEEERZhbxdaYFLL98I5n9_wQ
```

```bash
$ export POD2_SA_TOKEN=$(k exec -ti tester2 -- cat ${TOKEN_MOUNTED_DIR}/token)
```

---

Now, Pod list request is allowed:

```bash
$ k exec -ti tester2 -- curl -H "Authorization: Bearer $POD2_SA_TOKEN" -i https://kubernetes/api/v1/namespaces/default/pods --insecure
HTTP/2 200
cache-control: no-cache, private
content-type: application/json
date: Wed, 03 Mar 2021 03:51:21 GMT

{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces/default/pods",
    "resourceVersion": "42888"
  },
  "items": [
    {
      "metadata": {
        "name": "tester",
        "namespace": "default",
# Output omitted
    {
      "metadata": {
        "name": "tester2",
        "namespace": "default",
# Output omitted
```

---

### Cleanup

```bash
$ k delete role list-pods; k delete rolebinding list-pods_demo-sa; k delete pod tester; k delete pod tester2
role.rbac.authorization.k8s.io "list-pods" deleted
rolebinding.rbac.authorization.k8s.io "list-pods_demo-sa" deleted
pod "tester" deleted
pod "tester2" deleted
```

---

## References

* [Kubernetes Tips: Using a ServiceAccount](https://medium.com/better-programming/k8s-tips-using-a-serviceaccount-801c433d0023)
