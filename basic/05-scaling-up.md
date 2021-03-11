Prerequisites
==============

Minikube - a tool that lets you run Kubernetes locally. minikube runs a single-node Kubernetes cluster locally


The problem
============

Scale the application to keep up with user demand

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

2) scale echo app:

```
$ kubectl scale deployments/k8s-echo --replicas=4
deployment.apps/k8s-echo scaled

$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
k8s-echo              4/4     4            1           3m9s <-- note the delay after scale command is invoke - the creation of replicas requires some time
```

the next command shows the details behind scaled cluster:

```
$ kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
k8s-echo-5f58584b97-24mmc              1/1     Running   0          7m33s   172.18.0.5    minikube   <none>           <none>
k8s-echo-5f58584b97-k4tpk              1/1     Running   0          4m27s   172.18.0.13   minikube   <none>           <none>
k8s-echo-5f58584b97-qbxgx              1/1     Running   0          4m27s   172.18.0.12   minikube   <none>           <none>
k8s-echo-5f58584b97-z496m              1/1     Running   0          4m27s   172.18.0.11   minikube   <none>           <none>
```

3) Get description for what just happend:

```
$ kubectl describe deployments/k8s-echo
Name:                   k8s-echo
Namespace:              default
CreationTimestamp:      Wed, 10 Mar 2021 14:46:04 +0000
Labels:                 app=k8s-echo
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=k8s-echo
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=k8s-echo
  Containers:
   simple-web:
    Image:        yeasy/simple-web
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   k8s-echo-5f58584b97 (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  9m48s  deployment-controller  Scaled up replica set k8s-echo-5f58584b97 to 1
  Normal  ScalingReplicaSet  6m42s  deployment-controller  Scaled up replica set k8s-echo-5f58584b97 to 4
``` 

4) create a service:

```
$ kubectl expose deployment/k8s-echo --type="NodePort" --port 80
service/k8s-echo exposed

$ kubectl describe services/k8s-echo
Name:                     k8s-echo
Namespace:                default
Labels:                   app=k8s-echo
Annotations:              <none>
Selector:                 app=k8s-echo
Type:                     NodePort
IP:                       10.110.31.186
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30990/TCP
Endpoints:                172.18.0.11:80,172.18.0.12:80,172.18.0.13:80 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

5) test the service and check the default load-balancer strategy (round-robin):

```
$ minikube ip
172.17.0.57

$ curl http://172.17.0.57:30990
<!DOCTYPE html>
<html>
    <body>
        <center>
            <h1>
                <font color="blue" face="Georgia, Arial" size=8>
                    <em>Real</em>
                </font> Visit Results
            </h1>
        </center>
        <p style="font-size:150%" >#2021-03-10 15:01:50: 
            <font color="red">2</font> requests from &lt
            <font color="blue">LOCAL: 172.18.0.1</font>&gt to WebServer &lt
            <font color="blue">172.18.0.12</font>&gt
        </p>
    </body>
</html>

$ curl http://172.17.0.57:30990

<!DOCTYPE html>
<html>
    <body>
        <center>
            <h1>
                <font color="blue" face="Georgia, Arial" size=8>
                    <em>Real</em>
                </font> Visit Results
            </h1>
        </center>
        <p style="font-size:150%" >#2021-03-10 15:01:54: 
            <font color="red">2</font> requests from &lt
            <font color="blue">LOCAL: 172.18.0.1</font>&gt to WebServer &lt
            <font color="blue">172.18.0.11</font>&gt
        </p>
    </body>
</html>

$ curl http://172.17.0.57:30990

<!DOCTYPE html>
<html>
    <body>
        <center>
            <h1>
                <font color="blue" face="Georgia, Arial" size=8>
                    <em>Real</em>
                </font> Visit Results
            </h1>
        </center>
        <p style="font-size:150%" >#2021-03-10 15:01:56: 
            <font color="red">2</font> requests from &lt
            <font color="blue">LOCAL: 172.18.0.1</font>&gt to WebServer &lt
            <font color="blue">172.18.0.13</font>&gt
        </p>
    </body>
</html>

$ curl http://172.17.0.57:30990

<!DOCTYPE html>
<html>
    <body>
        <center>
            <h1>
                <font color="blue" face="Georgia, Arial" size=8>
                    <em>Real</em>
                </font> Visit Results
            </h1>
        </center>
        <p style="font-size:150%" >#2021-03-10 15:01:59: 
            <font color="red">2</font> requests from &lt
            <font color="blue">LOCAL: 172.18.0.1</font>&gt to WebServer &lt
            <font color="blue">172.18.0.5</font>&gt
        </p>
    </body>
</html>$ 
```

6) scale down operation could be performed in a similar way:

```
kubectl scale deployments/k8s-echo --replicas=2
```

