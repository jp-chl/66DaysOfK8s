# Day 66 of #66DaysOfK8s

_Last update: 2021-03-17_

---
Today, I have worked with Helm.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS ssh client
* kubectl client and server v1.19.0
* helm v3.2.4

---

## Setup

* Set an alias for kubectl (```alias k=kubectl```).
* Install Helm (```brew install helm```).

---

## Tasks

* Install a Helm Chart.
* Manage a release.

---

### Install a Helm Chart

Helm, a graduated CNCF project, simplifies an application deployment with templates (_charts_). A deployed chart is called a **release**.

Remote and local charts are supported. In order to use a remote Helm chart hosting, it is required to add the hosting repository. Locally, Helm supports custom charts creation.

---

For instance, Helm charts can be easily found at [Artifact Hub](https://artifacthub.io/) (a CNCF open source Website).

A nginx ingress controller can be installed with the next steps.

Go to Artifact Hub, and look for [```ingress-nginx```](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx).
Add repository and install as described.

```bash
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
"ingress-nginx" has been added to your repositories
```

```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "codecentric" chart repository
...Successfully got an update from the "es-operator" chart repository
...Successfully got an update from the "akomljen-charts" chart repository
...Successfully got an update from the "kubevious" chart repository
...Successfully got an update from the "prometheus-community" chart repository
...Successfully got an update from the "loki" chart repository
...Successfully got an update from the "grafana" chart repository
...Successfully got an update from the "bitnami" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

```bash
$ helm install my-ingress-nginx ingress-nginx/ingress-nginx --version 3.24.0
NAME: my-ingress-nginx
LAST DEPLOYED: XXX
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w my-ingress-nginx-controller'

An example Ingress that makes use of the controller:

  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

---

```bash
$ k get all
NAME                                               READY   STATUS    RESTARTS   AGE
pod/my-ingress-nginx-controller-694d5f5474-m5qmd   1/1     Running   0          34s

NAME                                            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/kubernetes                              ClusterIP      10.96.0.1       <none>        443/TCP                      2d1h
service/my-ingress-nginx-controller             LoadBalancer   10.109.65.214   <pending>     80:31014/TCP,443:30829/TCP   34s
service/my-ingress-nginx-controller-admission   ClusterIP      10.104.191.17   <none>        443/TCP                      34s

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-ingress-nginx-controller   1/1     1            1           34s

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/my-ingress-nginx-controller-694d5f5474   1         1         1       34s
```

---

## Upgrade a release

```bash
$ helm upgrade my-ingress-nginx ingress-nginx/ingress-nginx --set=controller.replicaCount=3
Release "my-ingress-nginx" has been upgraded. Happy Helming!
NAME: my-ingress-nginx
LAST DEPLOYED: Wed Mar 17 23:43:48 2021
NAMESPACE: default
STATUS: deployed
REVISION: 2 # Every release change spawns a new revision
TEST SUITE: None
# Output omitted
```

```bash
$ k get pods
NAME                                           READY   STATUS    RESTARTS   AGE
my-ingress-nginx-controller-694d5f5474-bgrv6   0/1     Running   0          7s
my-ingress-nginx-controller-694d5f5474-gz2r5   0/1     Running   0          7s
my-ingress-nginx-controller-694d5f5474-m5qmd   1/1     Running   0          5m34s
```

---

Helm allows a release rollback. To go back to revision 1:

```bash
$ helm rollback my-ingress-nginx 1
Rollback was a success! Happy Helming!
```

```bash
$ k get pods
NAME                                           READY   STATUS        RESTARTS   AGE
my-ingress-nginx-controller-694d5f5474-bgrv6   1/1     Terminating   0          4m38s
my-ingress-nginx-controller-694d5f5474-gz2r5   1/1     Terminating   0          4m38s
my-ingress-nginx-controller-694d5f5474-m5qmd   1/1     Running       0          10m
```

---

To list all revisions:

```bash
$  helm list
NAME            	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART               	APP VERSION
my-ingress-nginx	default  	3       	2021-03-17 23:48:27.603125 -0300 -03	deployed	ingress-nginx-3.24.0	0.44.0
```

---

To uninstall a release:

```bash
$ helm uninstall my-ingress-nginx; k get all
release "my-ingress-nginx" uninstalled

NAME                                               READY   STATUS        RESTARTS   AGE
pod/my-ingress-nginx-controller-694d5f5474-x9z9z   1/1     Terminating   0          2m7s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d2h
```

---

## References

* [Helm (official site)](https://helm.sh/docs/)
