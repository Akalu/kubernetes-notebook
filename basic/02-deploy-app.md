Prerequisites
==============
Minikube - a tool that lets you run Kubernetes locally. minikube runs a single-node Kubernetes cluster locally


The problem
============

Deploy a simple app to the node

Solution
=========

1) check the kurenetes is up and running and there is a cluster consisting from one node

```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.3", GitCommit:"06ad960bfd03b39c8310aaf92d1e7c12ce618213", GitTreeState:"clean", BuildDate:"2020-02-11T18:14:22Z", GoVersion:"go1.13.6", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.3", GitCommit:"06ad960bfd03b39c8310aaf92d1e7c12ce618213", GitTreeState:"clean", BuildDate:"2020-02-11T18:07:13Z", GoVersion:"go1.13.6", Compiler:"gc", Platform:"linux/amd64"}
```

```
$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   2m35s   v1.17.3
```

2) Create a deployment with name "k8s-echo" using image yeasy/simple-web

```
$ kubectl create deployment k8s-echo --image=yeasy/simple-web
deployment.apps/k8s-echo created

$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
k8s-echo              1/1     1            1           27m
```

3) start a k8s proxy on default local port 8001

```
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

4) check the proxy is accessible

```
$ curl http://localhost:8001/version

{
  "major": "1",
  "minor": "17",
  "gitVersion": "v1.17.3",
  "gitCommit": "06ad960bfd03b39c8310aaf92d1e7c12ce618213",
  "gitTreeState": "clean",
  "buildDate": "2020-02-11T18:07:13Z",
  "goVersion": "go1.13.6",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

5) list all pods

```
$ kubectl get pods
NAME                                   READY   STATUS             RESTARTS   AGE
k8s-echo-5f58584b97-vv98s              1/1     Running            0          28m
```






