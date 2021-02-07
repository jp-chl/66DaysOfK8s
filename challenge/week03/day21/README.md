# Day 21 of #66DaysOfK8s

_Last update: 2021-01-31_

---

Today, I have created a pod via Kubernetes API (without kubectl) securely with TLS.


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
* All pods are deployed on default namespace

---

## Tasks

* Get certificate and keys from existing minikube config file.
* Create a Pod by calling Kubernetes API.

---

### Get certificate and keys from existing minikube config file

By default, in order to make a request to any K8s cluster you use a config file (normally located in ```$HOME/.kube``` directory).

In this example, we'll be using a config file configured to connect to a local minikube K8s cluster, similar to this one:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/xxx/.minikube/ca.crt
    server: https://192.168.64.72:8443 # <----
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

To make an API call to K8s (instead of using kubectl) you can use its embedded Rest API. Kubectl transforms any command to its equivalent json format in order to call the Rest API.

Restful API can be called with curl. For instance, to get pods from default namespace you can call the ```/api/v1/pods``` endpoint:

```bash
$ curl https://<server-ip>:<server-port>/api/v1/pods
```

Let's get the K8s server, in this case https://192.168.64.72:8443.

```bash
export SERVER=$(grep server $HOME/.kube/config|cut -d" " -f 6)
```

```bash
$ echo $SERVER
https://192.168.64.72:8443
```

So, now we can call ```/api/v1/pods``` endpoint:

```bash
$ curl $SERVER/api/v1/pods
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

It failed because that endpoint requires the appropriate certificate and keys (located in ```config``` file).

---

Extract ```client-certificate```, ```client-key``` and ```certificate-authority``` from config file in order to create pods with a ```curl``` call.

```bash
export CLIENT=$(cat $(grep client-cert ./config.yaml|cut -d" " -f 6))
```

```bash
$ echo CLIENT
-----BEGIN CERTIFICATE-----
MIIDADCCAeigAwIBAgIBAjANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5p
# Output omitted
```

```bash
export KEY=$(cat $(grep client-key ./config.yaml|cut -d" " -f 6))
```

```bash
$ echo $KEY
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAwoiDBEIoVXPJJKZefXG3Mj/UNvJXh155aVWiIUcB6RrAQvlk
# Output omitted
```

```bash
export CERT=$(cat $(grep certificate-authority ./config.yaml|cut -d" " -f 6))
```

```bash
$ echo $CERT
-----BEGIN CERTIFICATE-----
MIIC5zCCAc+gAwIBAgIBATANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5p
# Output omitted
```

---

Save last variables to .pem files.

```bash
echo $CLIENT > ./client.pem
```

```bash
echo $KEY > ./client-key.pem
```

```bash
echo $AUTH > ./ca.pem
```

---

Now let's try to get pods again.

```bash
$ curl --cert ./client.pem --key ./client-key.pem --cacert ./ca.pem $SERVER/api/v1/pods
```

```json
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/pods",
    "resourceVersion": "212799"
  },
  "items": [
    {
      "metadata": {
```

```bash
# Output omitted
```

---

### Create a Pod by calling Kubernetes API

We'll be creating a simple nginx container as a Pod.

Create a json file with a Pod description.

```json
{
    "kind": "Pod",
    "apiVersion": "v1",
    "metadata": {
        "name": "curlpod",
        "namespace": "default",
        "labels": {
            "name": "curlpod"
        }
    },
    "spec": {
        "containers": [
            {
                "name": "curlpod",
                "image": "nginx",
                "ports": [{"containerPort": 80}]
            }
        ]
    }
}
```

---

Create a Pod by calling a POST request to ```/api/v1/namespaces/default/pods``` endpoint.

```bash
$ curl --cert ./client.pem --key ./client-key.pem --cacert ./ca.pem $SERVER/api/v1/namespaces/default/pods -X POST -H 'Content-Type: application/json' -d@curlpod.json
```

```json
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "curlpod",
    "namespace": "default",
    "selfLink": "/api/v1/namespaces/default/pods/curlpod",
    "uid": "863f7ffe-0deb-4101-8abe-0d2516f61418",
    "resourceVersion": "213350",
    "creationTimestamp": "2021-01-31T15:39:39Z",
    "labels": {
      "name": "curlpod"
    },
    "managedFields": [
      {
        "manager": "curl",
```

```bash
# Output omitted
```

---

List the newly created Pod with ```kubectl``` command.

```bash
$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
curlpod   1/1     Running   0          9s
```

## Cleanup

```bash
$ kubectl delete pod curlpod
pod "curlpod" deleted
```

---

## References

* [Kubernetes API Concepts (official site)](https://kubernetes.io/docs/reference/using-api/api-concepts/)
