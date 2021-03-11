Prerequisites
==============

Minikube - a tool that lets you run Kubernetes locally. minikube runs a single-node Kubernetes cluster locally


The problem
============
View running pods and nodes 

Solution
=========
Use the following commands to list and view resources:

* kubectl get - list resources
* kubectl describe - show detailed information about a resource
* kubectl logs - print the logs from a container in a pod
* kubectl exec - execute a command on a container in a pod

In greate details, command by command:

1) List all pods

```
$ kubectl get pods
NAME                                   READY   STATUS             RESTARTS   AGE
k8s-echo-5f58584b97-vv98s              1/1     Running            0          17m
```

2) Returns the details about each pod:

```
$ kubectl describe pods


Name:         k8s-echo-5f58584b97-vv98s
Namespace:    default
Priority:     0
Node:         minikube/172.17.0.107
Start Time:   Wed, 10 Mar 2021 12:34:49 +0000
Labels:       app=k8s-echo
              pod-template-hash=5f58584b97
Annotations:  <none>
Status:       Running
IP:           172.18.0.9
IPs:
  IP:           172.18.0.9
Controlled By:  ReplicaSet/k8s-echo-5f58584b97
Containers:
  simple-web:
    Container ID:   docker://8ea5211fb8091e7ea26ddcd2299ecfe9efc6d4ba3b60f50b2bf7b3c4f69d25a9
    Image:          yeasy/simple-web
    Image ID:       docker-pullable://yeasy/simple-web@sha256:356de309052fe233ba08eb4c9ad85ab89398f31555e8777326d57307ac913727
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 10 Mar 2021 12:36:05 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-9l822 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-9l822:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-9l822
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  18m   default-scheduler  Successfully assigned default/k8s-echo-5f58584b97-vv98s to minikube
  Normal  Pulling    18m   kubelet, minikube  Pulling image "yeasy/simple-web"
  Normal  Pulled     16m   kubelet, minikube  Successfully pulled image "yeasy/simple-web"
  Normal  Created    16m   kubelet, minikube  Created container simple-web
  Normal  Started    16m   kubelet, minikube  Started container simple-web
```

3) test that the deployed application is up and running

```
$ curl http://172.18.0.9:80

<!DOCTYPE html> <html> <body><center><h1><font color="blue" face="Georgia, Arial" size=8><em>Real</em></font> Visit Results</h1></center><p style="font-size:150%" >#2021-03-10 13:03:55: <font color="red">8</font> requests from &lt<font color="blue">LOCAL: 172.18.0.1</font>&gt to WebServer &lt<font color="blue">172.18.0.9</font>&gt</p></body> </html>$ 
```

NOTE: A simple-web apps is just a simple echo service which return its own ip address together with ip address of requester. Requests must be sent to port 80

A different way to do the same:

```
$ curl http://localhost:8001/api/v1/namespaces/default/pods/k8s-echo-5f58584b97-vv98s/proxy/
```

4) The following command shows the pod's logs: 

```
$ kubectl logs k8s-echo-5f58584b97-vv98s
```

5) And finally, one can execute common linux commands on running containers:

listing container's env variables:

```
$ kubectl exec k8s-echo-5f58584b97-vv98s env
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=k8s-echo-5f58584b97-vv98s
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
LANG=C.UTF-8
GPG_KEY=C01E1CAD5EA2C4F0B8E3571504C367C218ADD4FF
PYTHON_VERSION=2.7.14
PYTHON_PIP_VERSION=10.0.1
HOME=/root
```

attaching a console to container and viewing local fs with files (obviously one can execute any command, but the image itself is immutable):

```
$ kubectl exec -ti k8s-echo-5f58584b97-vv98s bash
root@k8s-echo-5f58584b97-vv98s:/code# ls
Dockerfile  README.md  index.html  index.py  pickle_data.txt
root@k8s-echo-5f58584b97-vv98s:/code# echo "writing output to file" > out.txt
```
