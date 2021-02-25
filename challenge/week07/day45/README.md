

Inject secrets into Pods


# Day 45 of #66DaysOfK8s

_Last update: 2021-02-24_

---
Today, I have worked injecting secrets into Pods.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* minikube: v1.13.0
* kubectl Client: v1.17.4
* kubectl Server: v1.19.0

---

## Creating secrets

You can create secrets either in imperative or in declarative way.

Imperatively, let's create a secret called ```my-secret-1``` with three fields (```secret1```, ```secret2``` and ```secret3```).

By default, the secret type is ```Opaque```.

```bash
$ kubectl create secret generic my-secret-1 --from-literal=mysecret1=data1 --from-literal=mysecret2=data2 --from-literal=mysecret3=data3
secret/my-secret-1 created
```

```bash
$ kubectl describe secret my-secret-1
Name:         my-secret-1
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
mysecret1:  5 bytes
mysecret2:  5 bytes
mysecret3:  5 bytes
```

---

## Injecting secrets into Pods

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-secret-pod
spec:
  containers:
    - name: secret-container
      image: nginx
      envFrom:
      - secretRef:
          name: my-secret-1
```

```bash
$ kubectl create -f yaml/pod.yaml
pod/my-secret-pod created
```

```bash
$ kubectl describe pod my-secret-pod
Name:         my-secret-pod
Namespace:    default
# Output omitted
Containers:
  secret-container:
    Container ID:
    Image:          nginx
# Output omitted
    Environment Variables from:
      my-secret-1  Secret  Optional: false
    Environment:   <none>
# Output omitted
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  8s    default-scheduler  Successfully assigned default/my-secret-pod to minikube
  Normal  Pulling    7s    kubelet, minikube  Pulling image "nginx"
```

Now, secrets have been injected into the Pod as environment variables.

```bash
$ kubectl exec -ti my-secret-pod -- env |grep mysecret
mysecret1=data1
mysecret2=data2
mysecret3=data3
```

