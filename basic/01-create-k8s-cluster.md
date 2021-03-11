Prerequisites
==============
Minikube - a tool that lets you run Kubernetes locally. minikube runs a single-node Kubernetes cluster locally

The problem
============
Create a cluster consisting from one node


Solution
=========

1) install minikube 

2) check the minikube is installed

```
$ minikube version
minikube version: v1.8.1
commit: cbda04cf6bbe65e987ae52bb393c10099ab62014
```

3) start a cluster from 1 node

```
$ minikube start
* minikube v1.8.1 on Ubuntu 18.04
* Using the none driver based on user configuration
* Running on localhost (CPUs=2, Memory=2460MB, Disk=145651MB) ...
* OS release is Ubuntu 18.04.4 LTS
* Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
  - kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
* Launching Kubernetes ... 
* Enabling addons: default-storageclass, storage-provisioner
* Configuring local host environment ...
* Waiting for cluster to come online ...
* Done! kubectl is now configured to use "minikube"
```

4) get information about cluster

```
$ kubectl cluster-info
Kubernetes master is running at https://172.17.0.69:8443
KubeDNS is running at https://172.17.0.69:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

5) get information about nodes

```
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   28s   v1.17.0
```

