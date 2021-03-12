About
======

This is a collection of recipes showing how to create, update and tune applications to be run on K8s cluster

Based on the various online tutorials (see the whole list of references in the References section)

Tools
=======

One can perform low-scale experiments with Kubernetes using minikube tool. Minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.

Requirements:

* 2 CPUs or more
* 2GB of free memory
* Container or virtual machine manager, such as: Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMWare


1) run docker

2) run minikube cluster from 1 node using the following command (this takes some time if to run the first time):

```
minikube start --memory 2048 --cpus=2

* minikube v1.18.1 on Microsoft Windows 10 Pro 10.0.19042 Build 19042
* Automatically selected the docker driver. Other choices: hyperv, ssh
* Starting control plane node minikube in cluster minikube
* Pulling base image ...
* Creating docker container (CPUs=2, Memory=2048MB) ...
* Preparing Kubernetes v1.20.2 on Docker 20.10.3 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v4
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

NOTE: if memory/cpu configuration has changed, this will require to re-create cluster using minikube delete command

After creating cluster check it is  up and running:

```
kubectl version

Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.7", GitCommit:"1dd5338295409edcfff11505e7bb246f0d325d15", GitTreeState:"clean", BuildDate:"2021-01-13T13:23:52Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"windows/amd64"}
Server Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.2", GitCommit:"faecb196815e248d3ecfb03c680a4507229c2a56", GitTreeState:"clean", BuildDate:"2021-01-13T13:20:00Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
```

Tesaurus
=========

A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a node is a cluster resource.

A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany, etc

A PodDisruptionBudget (PDB) limits the number of Pods of a replicated application that are down simultaneously from voluntary disruptions. For example, a quorum-based application would like to ensure that the number of replicas running is never brought below the number needed for a quorum.


References
===========

[1] https://kubernetes.io/docs/tutorials/kubernetes-basics/
[2] https://minikube.sigs.k8s.io/docs/handbook/
[3] https://minikube.sigs.k8s.io/docs/start/
[4] https://github.com/kubernetes-client/java
[5] https://github.com/kubernetes-client/python

