Prerequisites
==============

Minikube - a tool that lets you run Kubernetes locally, i.e. minikube runs a single-node Kubernetes cluster locally


The problem
============

* How to deploy a ZooKeeper ensemble using StatefulSet
* How to consistently configure the ensemble
* How to spread the deployment of ZooKeeper servers in the ensemble
* How to use PodDisruptionBudgets to ensure service availability during planned maintenance

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

3) create a ZooKeeper ensemble - the StatefulSet controller creates 1 Pod(s), and each Pod has a container with a ZooKeeper server. I downsized the number of Pods till 1 in order to run all smoothly on local machine with limited resources

```
$ kubectl apply -f ./files/zookeeper.yaml

service/zk-hs created
service/zk-cs created
poddisruptionbudget.policy/zk-pdb created
statefulset.apps/zk created
```

4) check the status of Pods:

```
$ kubectl get pods -w -l app=zk

NAME   READY   STATUS    RESTARTS   AGE
zk-0   0/1     Running   0          3s
zk-0   1/1     Running   0          19s
```

5) Sanity testing the ensemble

The most basic sanity test is to write data to one ZooKeeper server and to read the data from another.

The command below executes the zkCli.sh script to write world to the path /hello on the zk-0 Pod in the ensemble.

```
$ kubectl exec zk-0 -- zkCli.sh create /hello world

Connecting to localhost:2181
2021-03-12 15:25:03,915 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
2021-03-12 15:25:03,917 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=zk-0.zk-hs.default.svc.cluster.local
2021-03-12 15:25:03,917 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_131
2021-03-12 15:25:03,919 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2021-03-12 15:25:03,919 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-8-openjdk-amd64/jre
2021-03-12 15:25:03,919 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/usr/bin/../build/classes:/usr/bin/../build/lib/*.jar:/usr/bin/../share/zookeeper/zookeeper-3.4.10.jar:/usr/bin/../share/zookeeper/slf4j-log4j12-1.6.1.jar:/usr/bin/../share/zookeeper/slf4j-api-1.6.1.jar:/usr/bin/../share/zookeeper/netty-3.10.5.Final.jar:/usr/bin/../share/zookeeper/log4j-1.2.16.jar:/usr/bin/../share/zookeeper/jline-0.9.94.jar:/usr/bin/../src/java/lib/*.jar:/usr/bin/../etc/zookeeper:
2021-03-12 15:25:03,919 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib/x86_64-linux-gnu/jni:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu:/usr/lib/jni:/lib:/usr/lib
2021-03-12 15:25:03,920 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2021-03-12 15:25:03,920 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2021-03-12 15:25:03,920 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2021-03-12 15:25:03,920 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2021-03-12 15:25:03,920 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=5.4.72-microsoft-standard-WSL2
2021-03-12 15:25:03,920 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=zookeeper
2021-03-12 15:25:03,920 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/home/zookeeper
2021-03-12 15:25:03,921 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/
2021-03-12 15:25:03,922 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@22d8cfe0
2021-03-12 15:25:03,940 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
2021-03-12 15:25:03,991 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@876] - Socket connection established to localhost/127.0.0.1:2181, initiating session
2021-03-12 15:25:04,012 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x178270818600000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
Created /hello
```

To get the data from the zk-1 Pod use the following command.

```
$ kubectl exec zk-1 -- zkCli.sh get /hello

Connecting to localhost:2181
2021-03-12 15:26:55,923 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
2021-03-12 15:26:55,925 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=zk-0.zk-hs.default.svc.cluster.local
2021-03-12 15:26:55,925 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_131
2021-03-12 15:26:55,927 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2021-03-12 15:26:55,927 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-8-openjdk-amd64/jre
2021-03-12 15:26:55,927 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/usr/bin/../build/classes:/usr/bin/../build/lib/*.jar:/usr/bin/../share/zookeeper/zookeeper-3.4.10.jar:/usr/bin/../share/zookeeper/slf4j-log4j12-1.6.1.jar:/usr/bin/../share/zookeeper/slf4j-api-1.6.1.jar:/usr/bin/../share/zookeeper/netty-3.10.5.Final.jar:/usr/bin/../share/zookeeper/log4j-1.2.16.jar:/usr/bin/../share/zookeeper/jline-0.9.94.jar:/usr/bin/../src/java/lib/*.jar:/usr/bin/../etc/zookeeper:
2021-03-12 15:26:55,927 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib/x86_64-linux-gnu/jni:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu:/usr/lib/jni:/lib:/usr/lib
2021-03-12 15:26:55,927 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2021-03-12 15:26:55,927 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2021-03-12 15:26:55,927 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2021-03-12 15:26:55,928 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2021-03-12 15:26:55,928 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=5.4.72-microsoft-standard-WSL2
2021-03-12 15:26:55,929 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=zookeeper
2021-03-12 15:26:55,929 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/home/zookeeper
2021-03-12 15:26:55,929 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/
2021-03-12 15:26:55,931 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@22d8cfe0
2021-03-12 15:26:55,950 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
2021-03-12 15:26:56,003 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@876] - Socket connection established to localhost/127.0.0.1:2181, initiating session
2021-03-12 15:26:56,014 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x178270818600001, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
world
cZxid = 0x2
ctime = Fri Mar 12 15:25:04 UTC 2021
mZxid = 0x2
mtime = Fri Mar 12 15:25:04 UTC 2021
pZxid = 0x2
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```

The data that you created on zk-0 is available on all the servers in the ensemble.
