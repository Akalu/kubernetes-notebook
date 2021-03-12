Prerequisites
==============

Minikube - a tool that lets you run Kubernetes locally, i.e. minikube runs a single-node Kubernetes cluster locally

The problem
============

* Create and validate a Cassandra headless Service
* Use a StatefulSet to create a Cassandra ring
* Validate the StatefulSet
* Modify the StatefulSet
* Delete the StatefulSet and its Pods

Solution
=========

1) start docker, then minikube

```
minikube start --memory 2048 --cpus=2
```

2) check the cluster's accessability (this command should return the versions both for client and server):

```
$ kubectl version
```

3) Create a Service to track all Cassandra StatefulSet members:

```
$ kubectl apply -f ./files/cassandra-service.yaml
service/cassandra created
```

4) Validation should return the following text:

```
$ kubectl get svc cassandra

NAME        TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
cassandra   ClusterIP   None         <none>        9042/TCP   12s
```

5) Create a cassandra StatefulSet using the command:

```
$ kubectl apply -f ./files/cassandra-statefulset.yaml

statefulset.apps/cassandra created
storageclass.storage.k8s.io/fast created
```

6) Check all Pods are created (it takes some time for all Pods to be created, especially if small amount of memory has been chosen and few CPUs):

```
$ kubectl get statefulset cassandra

NAME        READY   AGE
cassandra   0/3     60s

$ kubectl get pods -l="app=cassandra"

NAME          READY   STATUS    RESTARTS   AGE
cassandra-0   0/1     Running   0          3m25s
cassandra-1   0/1     Running   0          86s

$ kubectl exec -it cassandra-0 -- nodetool status

Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  104.54 KiB  32           100.0%            b260a723-a7de-450b-87f5-4141e716586c  Rack1-K8Demo
UN  172.17.0.4  70.9 KiB   32           100.0%            2d74981e-e3fd-4ea6-bd92-9c400ca7aa35  Rack1-K8Demo
```

7) To delete all volumes and services use the following commands:

```
$ kubectl delete statefulset -l app=cassandra
statefulset.apps "cassandra" deleted

$ kubectl delete persistentvolumeclaim -l app=cassandra
persistentvolumeclaim "cassandra-data-cassandra-0" deleted
persistentvolumeclaim "cassandra-data-cassandra-1" deleted

$ kubectl delete service -l app=cassandra
service "cassandra" deleted
```



