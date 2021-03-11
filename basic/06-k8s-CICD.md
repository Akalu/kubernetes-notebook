Prerequisites
==============

Minikube - a tool that lets you run Kubernetes locally. minikube runs a single-node Kubernetes cluster locally


The problem
============

Perform a rolling update using kubectl


Solution
=========

1) deploy the simple echo application and see how many replicas exist:

```
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
k8s-echo              0/1     1            0           5s


$ kubectl get rs
NAME                             DESIRED   CURRENT   READY   AGE
k8s-echo-5f58584b97              1         1         0       16s
```

2) start the rolling update for the images - they will start update one by one:

```
kubectl set image deployments/k8s-echo k8s-echo=yeasy/simple-web:latest
```

