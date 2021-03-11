Prerequisites
==============
Minikube - a tool that lets you run Kubernetes locally. minikube runs a single-node Kubernetes cluster locally


The problem
============

Expose Kubernetes applications outside the cluster: create a Deployment, and then expose it publicly via a Service

Solution
=========

1) list the current Services from our cluster:

```
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   73s
```

2) expose app to external world

```
$ kubectl expose deployment/k8s-echo --type="NodePort" --port 80
service/k8s-echo exposed
```

3) check the service has been created (but still on internal network - this means kubernetes just reserved an address in its network topology):

```
$ kubectl get services
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
k8s-echo              NodePort    10.96.152.60    <none>        80:30297/TCP   4s
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP          5m24s  <-- this is a default internal K8s service which is basically an entry point to K8s cluster
```

The details can be retrieved via command:

```
$ kubectl describe services/k8s-echo

Name:                     k8s-echo
Namespace:                default
Labels:                   app=k8s-echo
Annotations:              <none>
Selector:                 app=k8s-echo
Type:                     NodePort
IP:                       10.96.152.60
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30297/TCP
Endpoints:                172.18.0.7:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

The Deployment created automatically a label for our Pod (app=k8s-echo)

One can create your own labels for fast access:

```
$ kubectl label pod k8s-echo-5f58584b97-jhc2m  app=v1 --overwrite <-- associate a string 'k8s-echo-5f58584b97-jhc2m' with 'app=v1' and pin it to pod/k8s-echo-5f58584b97-jhc2m
pod/k8s-echo-5f58584b97-jhc2m labeled
```

As a result one can label many pods with the same labels and then query them in a nice way, like so:

```
$ kubectl get pods -l app=v1
NAME                        READY   STATUS    RESTARTS   AGE
k8s-echo-5f58584b97-jhc2m   1/1     Running   0          32m
```


4) One can test that the service is available outside the cluster on $(minikube ip):$NodePort:

```
$ minikube ip
172.17.0.71

$ curl http://172.17.0.71:30297
```


6) delete service(s);

```
$ kubectl delete service -l app=v1
service "k8s-echo" deleted
```
