# Day 15 of #66DaysOfK8s

_Last update: 2021-01-25_

---

Today, I have worked in part 7 of a series of lessons in order to review the Kubernetes Architecture.
On this 7th day, a focus is on Pod Networking.

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* N/A (only theory)

---

## Setup

* N/A

---

## Tasks

Understand:

* Pod Networking

---

### Single IP per Pod

* A Pod is a group of one or more containers, with shared storage (data volumes).
In terms of networking, all containers within Pods share the same/unique IP address for each address family.

* All containers in a Pod share the same [network namespace](https://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/) including the IP address and network ports, they share the network namespace of a special container known as the "_[pause container](https://stackoverflow.com/questions/48651269/what-are-the-pause-containers#:~:text=The%20'pause'%20container%20is%20a,containers%20that%20join%20that%20pod.)_". It is used to obtain an IP address, then all the other containers within the Pod will use its network namespace.

* Inside a Pod, containers can communicate with one another using ```localhost```. They also share the same _system hostname_ (Pod's name).

* To communicate with each other, containers can use the loopback interfaces, interprocess communication (IPC, like [SystemV semaphores](https://www.softprayog.in/programming/system-v-semaphores) or [POSIX shared memory](https://www.softprayog.in/programming/interprocess-communication-using-posix-shared-memory-in-linux)) or write files on a common filesystem.

* As of 1.16 K8s version, IPv4 and IPv6 can be used for Pods and Services.

---

### Container Network

* Containers within Pods share the same namespace and IP address, configured by kube-proxy.

* Container's IP address is assigned even before their own start. This IP is set for the entire Pod lifecycle.

* A container will have an interface like _eth0@tun10_.

* Containers that want to interact with others running in a different Pod can use IP networking to communicate.

* A service connects the network traffic from a node high-number port to an endpoint (created at service start-up) using iptables and IPVS. The kube-controller-manager monitors if any endpoint/service is needed to create, update or delete.

---

## References

* [What is the role of 'pause' container? (Google groups)](https://groups.google.com/g/kubernetes-users/c/jVjv0QK4b_o?pli=1)

* [Differences between system V and Posix semaphores](
https://stackoverflow.com/questions/368322/differences-between-system-v-and-posix-semaphores)