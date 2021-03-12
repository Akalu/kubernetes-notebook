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

$ kubectl get rs
NAME                             DESIRED   CURRENT   READY   AGE
k8s-echo-5f58584b97              1         1         0       16s


$ kubectl describe pods
Name:         k8s-echo-5f58584b97-fhrj9
Namespace:    default
Priority:     0
Node:         minikube/172.17.0.37
Start Time:   Thu, 11 Mar 2021 07:57:18 +0000
Labels:       app=k8s-echo
              pod-template-hash=5f58584b97
Annotations:  <none>
Status:       Running
IP:           172.18.0.10
IPs:
  IP:           172.18.0.10
Controlled By:  ReplicaSet/k8s-echo-5f58584b97
Containers:
  simple-web:
    Container ID:   docker://661b80eb15ed1f3d109d9d55d565946e6089cbb25ba7ef6048d3eaa919f87b52
    Image:          yeasy/simple-web <-- THIS IS THE NAME OF THE CURRENT APPLICATION
    Image ID:       docker-pullable://yeasy/simple-web@sha256:356de309052fe233ba08eb4c9ad85ab89398f31555e8777326d57307ac913727
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 11 Mar 2021 07:57:54 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fjpkr (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-fjpkr:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fjpkr
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  64s   default-scheduler  Successfully assigned default/k8s-echo-5f58584b97-fhrj9 to minikube
  Normal  Pulling    63s   kubelet, minikube  Pulling image "yeasy/simple-web"
  Normal  Pulled     29s   kubelet, minikube  Successfully pulled image "yeasy/simple-web"
  Normal  Created    28s   kubelet, minikube  Created container simple-web
  Normal  Started    28s   kubelet, minikube  Started container simple-web <-- NOTE THE NAME OF THE CONTAINER

```

2) start the rolling update for the images - they will start update one by one:

```
$ kubectl set image deployments/k8s-echo simple-web=yeasy/simple-web:latest
deployment.apps/k8s-echo image updated

$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
k8s-echo-55478fbfbb-gqh8m              1/1     Running       0          9s       <-- all requests will be routed to this instance
k8s-echo-5f58584b97-fhrj9              1/1     Terminating   0          21m      <-- still running but not accessible
```
One can check that the instances have been rolled up:

```
$ kubectl rollout status deployments/k8s-echo
deployment "k8s-echo" successfully rolled out
```

