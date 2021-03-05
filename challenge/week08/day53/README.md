# Day 53 of #66DaysOfK8s

_Last update: 2021-03-04_

---
Today, I have worked with the Readiness probe.

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
* Helm: v3.2.4

---

## Tasks

* Configure a readiness probe for a Pod
* Set a not-ready status on Pod

---

## Configure a readiness probe for a Pod

_"The kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers."_ -- (official site)

The kubelet will check for liveness and readiness probe after the container has started.

A typical readiness probe looks like:

```yaml
apiVersion: v1
kind: Pod
# Output omitted
spec:
  containers:
  - name: readiness
    image: k8s.gcr.io/readiness
# Output omitted
    readinessProbe:
      httpGet:
        path: /health-endpoint # Container's endpoint to check readiness
        port: 8080
      initialDelaySeconds: 3 # Initial delay before periodically check readiness
      periodSeconds: 3 # Periodicity check
```

Where the endpoint is defined in ```path``` and ```port``` variables. Besides, a startup delay in seconds can be set by changing ```initialDelaySeconds```. K8s will test readiness every ```periodSeconds```.

---

Normally, readiness probe checks whether the Pod is operational and ready to receive traffic. For instance, if a microservice is linked to a database, a good practice is to do a Ping to the DB in every readiness call.

In the following example, we'll be creating a local MongoDB database and a Pod that uses it.

Let's start a local Mongo DB with a database called ```admin``` with credentials: ```user``` and ```secretpassword```.

```bash
$ helm install my-release --set auth.rootPassword=secretpassword,auth.username=my-user,auth.password=my-password,auth.database=my-database bitnami/mongodb
NAME: my-release
LAST DEPLOYED: Thu Mar  4 23:22:55 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

MongoDB(R) can be accessed on the following DNS name(s) and ports from within your cluster:

    my-release-mongodb.default.svc.cluster.local

To get the root password run:

    export MONGODB_ROOT_PASSWORD=$(kubectl get secret --namespace default my-release-mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)

To get the password for "my-user" run:

    export MONGODB_PASSWORD=$(kubectl get secret --namespace default my-release-mongodb -o jsonpath="{.data.mongodb-password}" | base64 --decode)

To connect to your database, create a MongoDB(R) client container:

    kubectl run --namespace default my-release-mongodb-client --rm --tty -i --restart='Never' --env="MONGODB_ROOT_PASSWORD=$MONGODB_ROOT_PASSWORD" --image docker.io/bitnami/mongodb:4.4.4-debian-10-r0 --command -- bash

Then, run the following command:
    mongo admin --host "my-release-mongodb" --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/my-release-mongodb 27017:27017 &
    mongo --host 127.0.0.1 --authenticationDatabase admin -p $MONGODB_ROOT_PASSWORD
```

```bash
$ echo kubectl get secret --namespace default my-release-mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 --decode
secretpassword
```

---

Now deploy a Pod to connect to the database. In this case, the mongo endpoint is ```mongodb://my-release-mongodb:27017```, database is ```admin```, user is ```root``` and password is ```secretpassword```.

Within microservice's code, the readiness endpoint is coded as:

```javascript
 @Get('readiness')
 @HealthCheck()
 checkReadiness(): Promise<HealthCheckResult> {
   return this.health.check([
     (): Promise<HealthIndicatorResult> =>
       this.mongooseHealth.pingCheck('mongodb'),
   ]);
 }
```

Deploy the Pod and check its log.

```yaml
# deploy.yaml
apiVersion: apps/v1
kind: Deployment
# Output omitted
spec:
# Output omitted
    spec:
      containers:
      - image: xxx
# Output omitted
        readinessProbe:
          httpGet:
            path: /api/v1/health/readiness
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 10
```

```bash
$ k apply -f yaml/.
configmap/my-config-map created
deployment.apps/readiness-test created
secret/my-secret-map created
```

```log
$ k logs -f pod/readiness-test-76d6f6dc5-6bt77
[NestWinston] Debug	3/5/2021, 11:06:37 AM [HealthController] checkReadiness [GET to /api/v1/health/readiness] - INIT - {}
[NestWinston] Debug	3/5/2021, 11:06:37 AM [HealthController] checkReadiness [GET to /api/v1/health/readiness] - END 1 ms - {}
```

---

## Set a not-healthy status on Pod

If we shut down the DB for a while, the readiness probe will start to fail. If we start the DB again, the readiness probe will succeed.

```bash
$ helm uninstall my-release
release "my-release" uninstalled
```

```log
$ k logs -f pod/readiness-test-76d6f6dc5-6bt77
# Output omitted
[NestWinston] Error	3/5/2021, 11:08:37 AM [HealthCheckService] Health Check has failed! {"mongodb":{"status":"down"}} - {"trace":""}
[NestWinston] Debug	3/5/2021, 11:08:40 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - INIT - {}
[NestWinston] Debug	3/5/2021, 11:08:40 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - END 18 ms - {}
# Output omitted
[NestWinston] Debug	3/5/2021, 11:09:07 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - INIT - {}
[NestWinston] Debug	3/5/2021, 11:09:07 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - END 8 ms - {}
[NestWinston] Debug	3/5/2021, 11:09:07 AM [HealthController] checkReadiness [GET to /api/v1/health/readiness] - INIT - {}
[NestWinston] Error	3/5/2021, 11:09:07 AM [HealthCheckService] Health Check has failed! {"mongodb":{"status":"down"}} - {"trace":""}
[NestWinston] Debug	3/5/2021, 11:09:10 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - INIT - {}
[NestWinston] Debug	3/5/2021, 11:09:10 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - END 1 ms - {}
# Output omitted
[NestWinston] Debug	3/5/2021, 11:09:37 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - INIT - {}
[NestWinston] Debug	3/5/2021, 11:09:37 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - END 7 ms - {}
[NestWinston] Debug	3/5/2021, 11:09:37 AM [HealthController] checkReadiness [GET to /api/v1/health/readiness] - INIT - {}
[NestWinston] Error	3/5/2021, 11:09:37 AM [HealthCheckService] Health Check has failed! {"mongodb":{"status":"down"}} - {"trace":""}
[NestWinston] Debug	3/5/2021, 11:09:40 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - INIT - {}
[NestWinston] Debug	3/5/2021, 11:09:40 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - END 1 ms - {}
# Output omitted
[NestWinston] Debug	3/5/2021, 11:10:07 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - INIT - {}
[NestWinston] Debug	3/5/2021, 11:10:07 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - END 4 ms - {}

# DB is operational once again
[NestWinston] Debug	3/5/2021, 11:10:07 AM [HealthController] checkReadiness [GET to /api/v1/health/readiness] - INIT - {}
[NestWinston] Debug	3/5/2021, 11:10:07 AM [HealthController] checkReadiness [GET to /api/v1/health/readiness] - END 1 ms - {}

[NestWinston] Debug	3/5/2021, 11:10:10 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - INIT - {}
[NestWinston] Debug	3/5/2021, 11:10:10 AM [HealthController] checkMemory [GET to /api/v1/health/liveness] - END 1 ms - {}
```

---

## Cleanup

```bash
$ helm uninstall my-release
release "my-release" uninstalled
```

```bash
$ k apply -f yaml/.
configmap "my-config-map" deleted
deployment.apps "readiness-test" deleted
secret "my-secret-map" deleted
```

---

## References

* [Configure Liveness, Readiness and Startup Probes (official site)](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
* [Bitnami MongoDB Helm Chart](https://github.com/bitnami/charts/tree/master/bitnami/mongodb/#installing-the-chart)
