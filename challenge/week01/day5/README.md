# Day 5 of #66DaysOfK8s

_Last update: 2021-01-15_

---

Today, I have started a series of lessons to create a K8s cluster from scratch in GCP.
In this lesson we will create two VMs, for the master and worker nodes (respectively).

#kubernetes #learning #K8s #66DaysChallenge

---

## Versions used

* macOS Catalina 10.15.7
* Google Chrome 87.0.4280.88 (Official Build) (x86_64)
* Native macOS clients for ssh-keygen and ssh

---

## Setup

* I'm using the GCP free tier.

---

## Tasks

* Create ssh keys for master/worker connectivity
* Create a VPC
* Create firewall rule
* Create a VM instance for master node
* Create a VM instance for worker node

---

## Create a private key

In Mac you can use ssh-keygen (in Windows, for example, Putty)

```bash
ssh-keygen -t rsa -b 4096 -C "student"
```

* "```-C```" specifies the user for our VM instances.
* Specifies a folder to save the keys (including the key name). I'm using "```K8sPK1```".
* Press enter for passphrase (default settings).

Save private (file no extension) and public (.pub) keys. We'll be using them later on.

---

## Create a GCP account

For this lab, I'm using the free tier.

![Cluster Monitoring Dashboard](readme-images/01.png)
![Cluster Monitoring Dashboard](readme-images/02.png)
![Cluster Monitoring Dashboard](readme-images/03.png)

---

## Create a Project

I'm choosing "MyK8s" as the project name.

![Cluster Monitoring Dashboard](readme-images/04.png)
![Cluster Monitoring Dashboard](readme-images/05.png)

---

## Create a VPC

Go to the "VPC Network" option.

![Cluster Monitoring Dashboard](readme-images/06.png)

If this is your first time, you might have to enable "Google Engine API".

![Cluster Monitoring Dashboard](readme-images/07.png)

You will see the "default" VPC.

![Cluster Monitoring Dashboard](readme-images/08.png)

Press the "Create VPC Network" button.

![Cluster Monitoring Dashboard](readme-images/10.png)

Complete with:
* VPC name: k8svpc1
* New subnet:
  * name: k8svpc1
  * region: us-central1
  * IP address range: 10.2.0.0/16

![Cluster Monitoring Dashboard](readme-images/10a.png)

Press the "Create" button"

![Cluster Monitoring Dashboard](readme-images/11.png)

---

## Create a firewall rule

You don't have to wait for VPC creation while you can create a Firewall rule.
Go to Firewall menu and press "Create firewall rule" button.

![Cluster Monitoring Dashboard](readme-images/12.png)
![Cluster Monitoring Dashboard](readme-images/13.png)

Complete with:
* Name: k8sfr1
* Network: k8svpc1
* Targets: All instances in the network
* Source IP ranges: 0.0.0.0/0
* Protocols and ports: Allow all

Press the "Create" button"

![Cluster Monitoring Dashboard](readme-images/14.png)
![Cluster Monitoring Dashboard](readme-images/15.png)
![Cluster Monitoring Dashboard](readme-images/16.png)
![Cluster Monitoring Dashboard](readme-images/17.png)
![Cluster Monitoring Dashboard](readme-images/18.png)
![Cluster Monitoring Dashboard](readme-images/19.png)


Finally, select the VPC and press the "Enable API" button within "DNS server policy".

![Cluster Monitoring Dashboard](readme-images/20.png)
![Cluster Monitoring Dashboard](readme-images/23.png)
![Cluster Monitoring Dashboard](readme-images/24.png)

---

## Create a Master VM instance

Now, go to the "Compute Engine" menu, select "VM instances" and press "Create" button.

![Cluster Monitoring Dashboard](readme-images/25.png)
![Cluster Monitoring Dashboard](readme-images/26.png)

Complete with:
* Name: master
* Region: us-central1 (same as the vpc)
* Machine configuration:
  * Series: N1
  * Machine type: n1-standard-2 (2 vCPU, 7.5 GB memory)
* Boot disk:
  * Operating system: Ubuntu
  * Version: Ubuntu 18.04 LTS
  * Size: 20 (GB)
  * _click in "select" button_
* Management, security, disks, networking, sole tenancy:
  * Under "Security" tab, add the public SSK key (.pub file saved before). You can notice the "student" word comment.
  * Networking: k8svpc1

Press "Create" button.

![Cluster Monitoring Dashboard](readme-images/27.png)
![Cluster Monitoring Dashboard](readme-images/28.png)
![Cluster Monitoring Dashboard](readme-images/29.png)
![Cluster Monitoring Dashboard](readme-images/30.png)
![Cluster Monitoring Dashboard](readme-images/31.png)
![Cluster Monitoring Dashboard](readme-images/32.png)
![Cluster Monitoring Dashboard](readme-images/33.png)
![Cluster Monitoring Dashboard](readme-images/34.png)
![Cluster Monitoring Dashboard](readme-images/35.png)
![Cluster Monitoring Dashboard](readme-images/36.png)
![Cluster Monitoring Dashboard](readme-images/37.png)
![Cluster Monitoring Dashboard](readme-images/38.png)
![Cluster Monitoring Dashboard](readme-images/39.png)

---

## Connect to the master instance

Copy the "External IP" address of the latter created VM instance (master).

Connect to master instance:

```bash
$ ssh -i "K8sPK1" student@<external-master-ip>

...

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

student@master:~$
```

---

## Create a Worker VM instance

Select the, already created, master instance, and then press "Create similar" button.

![Cluster Monitoring Dashboard](readme-images/41.png)

Complete with:
* Name: worker
* Check all settings are similar with the master ones.

Press "Create" button.

![Cluster Monitoring Dashboard](readme-images/42.png)
![Cluster Monitoring Dashboard](readme-images/44.png)
---

## Connect to the worker instance

Copy the "External IP" address of the latter created VM instance (worker).

Connect to worker instance:

```bash
$ ssh -i "K8sPK1" student@<external-worker-ip>

...

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

student@worker:~$
```
