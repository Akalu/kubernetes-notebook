Prerequisites
==============

Minikube - a tool that lets you run Kubernetes locally, i.e. minikube runs a single-node Kubernetes cluster locally


The problem
============

* How to create a StatefulSet
* How a StatefulSet manages its Pods
* How to delete a StatefulSet
* How to scale a StatefulSet
* How to update a StatefulSet's Pods

Solution
=========

## Creating a StatefulSet

1) start docker, then minikube

```
minikube start --memory 2048 --cpus=2
```

2) check the cluster's accessability (this command should return the versions both for client and server):

```
$ kubectl version
```

3) In the first terminal one can watch the creation of the StatefulSet's Pods (this command indefinitely queries the status of pods, omit -w to get immediate output).

```
$ kubectl get pods -w -l app=nginx
```

The output will be like so (notice that the web-1 Pod is not launched until the web-0 Pod is Running and Ready):

```
NAME    READY   STATUS    RESTARTS   AGE
web-0   0/1     Pending   0          0s
web-0   0/1     Pending   0          0s
web-0   0/1     Pending   0          2s
web-0   0/1     ContainerCreating   0          2s
web-0   1/1     Running             0          27s
web-1   0/1     Pending             0          0s
web-1   0/1     Pending             0          0s
web-1   0/1     Pending             0          3s
web-1   0/1     ContainerCreating   0          3s
web-1   1/1     Running             0          4s
```

4) In the second terminal, use kubectl apply to create the headless Service and StatefulSet defined in web.yaml.

```
$ kubectl apply -f ./files/web.yaml
```

The configuration specified in web.yaml file creates two Pods, each running an nginx webserver

5) Get additional information:

```
$ kubectl get service nginx
NAME    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   None         <none>        80/TCP    5m24s

$ kubectl get statefulset web
NAME   READY   AGE
web    2/2     5m35s
```

## Getting full information about Pods

Pods in a StatefulSet have a unique ordinal index and a stable network identity

```
kubectl get pods -l app=nginx
```

One can spin up a standard networking toolkit to examine virtual network. For that use kubectl run to execute a container that provides the nslookup command from the dnsutils package:

```
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
```

This command starts a new shell. In that new shell, run:

```
# nslookup web-0.nginx

Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
Name:      web-0.nginx
Address 1: 172.17.0.3 web-0.nginx.default.svc.cluster.local

# nslookup web-1.nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
Name:      web-1.nginx
Address 1: 172.17.0.4 web-1.nginx.default.svc.cluster.local
```

## Writing to Stable Storage

```
$ kubectl get pvc -l app=nginx

NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-web-0   Bound    pvc-893db1e8-02c7-49ef-9d1e-39459ed9c8b2   1Gi        RWO            standard       94m
www-web-1   Bound    pvc-21628e73-310b-44cb-8feb-2b0deebe89ab   1Gi        RWO            standard       93m
```

One can write something to nginx' public directory to see the static content  served through request:

```
$ kubectl exec web-0 chmod 755 /usr/share/nginx/html

$ kubectl exec "web-0" -- sh -c "echo test_content > /usr/share/nginx/html/index.html"

$ kubectl exec web-0 curl 172.17.0.3

kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0test_content
100    13  100    13    0     0  20569      0 --:--:-- --:--:-- --:--:-- 13000
```

## Deleting all Pods

Note: this command is blocking and it can take some time to finish
```
$ kubectl delete pod -l app=nginx
```
