# Day 23 of #66DaysOfK8s

_Last update: 2021-02-02_

---

Today, I have worked with K8s Rest API based on a token provided by different K8s secrets.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Setup

* A minikube local cluster has to be already configured and running.
* The default config file (```$HOME/.kube``` directory has only one cluster configured, i.e. minikube)
* All tests run on minikube.

---

## Tasks

* Get a token from different secrets.
* Call different K8s Rest APIs by using a token.

---

### Get a token from different secrets

By default, in order to make a request to any K8s cluster you use a config file (normally located in ```$HOME/.kube``` directory).

In this example, we'll be using a config file configured to connect to a local minikube K8s cluster, similar to this one:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/xxx/.minikube/ca.crt
    server: https://192.168.64.72:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/xxx/.minikube/profiles/minikube/client.crt
    client-key: /Users/xxx/.minikube/profiles/minikube/client.key
```

To make an API call to K8s (instead of using kubectl) you can use its embedded Rest API. Kubectl transform any command to its equivalent json format in order to call the Rest API.

Restful API can be called with curl. For instance, to get pods from default namespace you can call the ```/api/v1/pods``` endpoint:

```bash
$ curl https://<server-ip>:<server-port>/api/v1/pods
```

Let's get the K8s server, in this case https://192.168.64.72:8443.

```bash
export SERVER=$(grep server $HOME/.kube/config|cut -d" " -f 6)
```
> _If you have more than one cluster configured in config file, you might have to change last export._

```bash
$ echo $SERVER
https://192.168.64.72:8443
```

If we call ```/api/v1/``` endpoint, we'll getting an error.

```bash
$ curl $SERVER/api/v1
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

---

By default, K8s secrets objects is associated with a token and a service account. For instance, let's look at two of them: ```default-token``` (in default namespace) and ```namespace-controller-token``` (in kube-system namespace).

```bash
$ kubectl get secrets -A
NAMESPACE         NAME                                             TYPE                                  DATA   AGE
default           default-token-wgsnn                              kubernetes.io/service-account-token   3      13m
kube-node-lease   default-token-t2kw5                              kubernetes.io/service-account-token   3      13m
kube-public       default-token-pbhbf                              kubernetes.io/service-account-token   3      13m
kube-system       attachdetach-controller-token-4w87q              kubernetes.io/service-account-token   3      13m
# Output omitted
kube-system       namespace-controller-token-djggm                 kubernetes.io/service-account-token   3      13m
```

```bash
$ kubectl -n default describe secret default-token-wgsnn
Name:         default-token-wgsnn
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 0fcc335b-5a46-416b-868b-d67b6747e468

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImRuSFNya0RFbm9TaGJVWDl0dE5PSFBPNW5SSE1uVzNSbDJqTUNieC1sSzAifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4td2dzbm4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBmY2MzMzViLTVhNDYtNDE2Yi04NjhiLWQ2N2I2NzQ3ZTQ2OCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.A4phnowvcKND-2fsuQRhWfJP65kvYdhyNWvWd-xiZl7wl13VTM_5mzpOZs3fgBV5xPdEqWrHeSSYfKx2RB6pmplTaz_HZ9WL8E5mNO1BThWXH3GMdqy28re23lzPTZ9RioJGb1csumgcRjp_CVeYfiI0DbydmQHQV9Kb5juNEOP-iO_dIB6hM8tdDkBpdbaEnKXXBBXL9UAckJcsOAb0mc35kKeIpLB_Ku9BItFQ9162cHPMP1t7j7wnDvpoNZwttC5DAtk2dY_kexHdTjKcjUQsGGhfAZ-_uolykgrlEaTWADbd_U6a8P0XV5OoMR1z3SvM5t0F0LpK-xq_IP_xvA
```

```bash
$ kubectl -n kube-system describe secret namespace-controller-token-djggm
Name:         namespace-controller-token-djggm
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: namespace-controller
              kubernetes.io/service-account.uid: 3c703adf-1112-46d5-a673-eef0a3c05758

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImRuSFNya0RFbm9TaGJVWDl0dE5PSFBPNW5SSE1uVzNSbDJqTUNieC1sSzAifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi1kamdnbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjNjNzAzYWRmLTExMTItNDZkNS1hNjczLWVlZjBhM2MwNTc1OCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.hEUe3CT-jD_JiHLbVQD8eosDTtt53DlRicyr4rHcrGh2us6xOXVGfe87vBwmQ9NmOZ40eK72BJW-bJa--dfLBkhONw3C5_ySY_7aeXLE23vSuh1MbqAUhVWeiX-UQnXx9Nix-A42DBgp-CPZfc-RhTAuVGyFlu4SXs9n3PvCSiEf-yQ24X3ftLZtKc25CgVv88qNLO3Mrxt_kcuOTnOt2933Hn3qXdDkek8-cHhUfK8C1KHe4KxWSYku_j73NjqmxTNKZRBoli-H2xhsir3CcmoNjzONFSNvjV18aaJ39uO7ItS98A4rYlHIjjZNFCNTcw2tGplHmlZRXYkgmplYbQ
```

---

Now, grab the token from each secret.

```bash
$ export TOKEN_DEFAULT=$(kubectl -n default describe secret default-token-wgsnn|grep ^token |cut -f7 -d ' ')
```

```bash
$ export TOKEN_NAMESPACE=$(kubectl -n kube-system describe secret namespace-controller-token-djggm|grep ^token |cut -f7 -d ' ')
```

---

### Call different K8s Rest APIs by using a token

```bash
$ curl $SERVER/api/v1 -H "Authorization: Bearer ${TOKEN_DEFAULT}" -k
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
```

---

```bash
$ curl $SERVER/api/v1 -H "Authorization: Bearer ${TOKEN_NAMESPACE}" -k
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
```

---

```bash
$ curl $SERVER/api/v1/namespaces -H "Authorization: Bearer ${TOKEN_DEFAULT}" -k
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "namespaces is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"namespaces\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "namespaces"
  },
  "code": 403
}
```

```bash
$ curl $SERVER/api/v1/namespaces -H "Authorization: Bearer ${TOKEN_NAMESPACE}" -k
{
  "kind": "NamespaceList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces",
    "resourceVersion": "1513"
  },
  "items": [
    {
      "metadata": {
        "name": "default",
        "selfLink": "/api/v1/namespaces/default",
        "uid": "4a7319a4-d49a-4149-abbc-d77002bb047d",
        "resourceVersion": "156",
        "creationTimestamp": "2021-02-02T22:29:19Z",
        "managedFields": [
          {
            "manager": "kube-apiserver",
# Output omitted
```

---

## Cleanup

```bash
$ kubectl delete pod busybox
pod "busybox" deleted
```

---

## References

* [Kubernetes API Concepts (official site)](https://kubernetes.io/docs/reference/using-api/api-concepts/)
