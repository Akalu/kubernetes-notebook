Prerequisites
==============
Minikube - a tool that lets you run Kubernetes locally, i.e. minikube runs a single-node Kubernetes cluster locally


The problem
============

Externalize application's configuration using MicroProfile, ConfigMaps and Secrets:

* Create a Kubernetes ConfigMap and Secret
* Inject microservice configuration using MicroProfile Config

Solution
=========

1) There are two web apps: inventory and system. 

The first one (inventory) exposes api to work with system resources of type SystemData, which is defined as follows:

```
class SystemData {
    String hostname;
    Properties properties;
}
```

The api is defined as follows:

GET /systems/{hostname}

{
    "os.name" : "Ubuntu 20.04 LTS"
}


The second application (system) exposes api with one GET endpoint localhost:9080 which returns the map of system properties of local machine:

GET /properties

{
    "os.name" : "Ubuntu 20.04 LTS"
}

The 1st microservice uses the 2nd one to query system data and present them in readable way.

```
<properties>
	<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!-- Default test properties -->
        <cluster.ip>localhost</cluster.ip>
        <system.node.port>31000</system.node.port>
        <system.appName>system</system.appName>
        <!-- Ports configuration -->
        <http.port>9080</http.port>
        <https.port>9443</https.port>
</properties
```

2) build images using Dockerfile

```
mvn package -pl inventory
mvn package -pl system
```

3) create containers using config kubernetes.yaml:

```
$ kubectl apply -f kubernetes.yaml
deployment.apps/system-deployment created
deployment.apps/inventory-deployment created
service/system-service created
service/inventory-service created

```

4) Test the system app:

```
$ curl -u user:pwd http://$( minikube ip ):31000/system/properties

{
  "java.vendor": "AdoptOpenJDK",
  "default.https.port": "9443",
  "os.name": "Linux",
  "file.encoding": "UTF-8",
  "java.specification.version": "1.8",
}

```

If to call inventory service, it will query the system service and forward response back (caching intermediate results in structure):

```
$ curl http://$( minikube ip ):32000/inventory/systems/system-service

{
  "java.vendor": "AdoptOpenJDK",
  "default.https.port": "9443",
  "os.name": "Linux",
  "file.encoding": "UTF-8",
  "java.specification.version": "1.8",
}

```

5) These apps use system env variables  defined as follows:

```
  // Basic Auth Credentials
  @Inject
  @ConfigProperty(name = "SYSTEM_APP_USERNAME")
  private String username;

  @Inject
  @ConfigProperty(name = "SYSTEM_APP_PASSWORD")
  private String password;
```

6) ConfigMap and Secrets can be setup through console:

```
$ kubectl create configmap sys-app-name --from-literal name=my-system
configmap/sys-app-name created

$ kubectl create secret generic sys-app-credentials --from-literal username=user --from-literal password=pwd
secret/sys-app-credentials created
```

7) After changes made it is necessary to rebuild and redeploy the applications for your changes to take effect. 
Rebuild the application using the following commands, making sure you're in the start directory:

```
mvn package -pl system
mvn package -pl inventory
```
