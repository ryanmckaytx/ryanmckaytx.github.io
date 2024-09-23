Title: Docker Java Example Part 5: Kubernetes
Date: 2017-09-14 17:28
Author: Ryan McKay
Tags: docker, kubernetes
Slug: docker-java-example-part-5-kubernetes
Status: draft
Summary: Now that I've got my project getting packaged up in a docker image, the next step in this POC is to look at platforms for running docker. The only PaaS that I am familiar with now is Pivotal Cloud Foundry, which we used at my last job to deploy Spring Boot executable jars. PCF was working on a docker story, not sure how far that got. It looks like they are pretty [bought into Kubernetes](https://pivotal.io/pks) these days. 

<div class="toc" markdown="1">
<span class="toctitle">Docker Java Example Series</span>

1. [Initializing a new Spring Boot Project](http://againstentropy.blogspot.com/2017/07/docker-java-example-part-1-initializing.html)
2. [Spring Web MVC Testing](http://againstentropy.blogspot.com/2017/08/docker-java-example-part-2-spring-web.html)
3. [Transmode Gradle Plugin](http://againstentropy.blogspot.com/2017/08/docker-java-example-part3-transmode-gradle-plugin.html)
4. [Bmuschko and Nebula Gradle Plugins](http://againstentropy.blogspot.com/2017/09/docker-java-example-part-4-bmuschko-nebula-gradle-docker-plugins.html)
5. [Kubernetes](https://againstentropy.blogspot.com/2017/09/docker-java-example-part-5-kubernetes.html)
</div>

Now that I've got my project getting packaged up in a docker image, the next step in this POC is to look at platforms for running docker. The only PaaS that I am familiar with now is Pivotal Cloud Foundry, which we used at my last job to deploy Spring Boot executable jars. PCF was working on a docker story, not sure how far that got. It looks like they are pretty [bought into Kubernetes](https://pivotal.io/pks) these days. In fact it seems like the whole cloud world is moving in that direction, with the likes of Pivotal, VMware, Amazon, Microsoft, Dell, Alibaba, and Mesosphere joining the [Cloud Native Computing Foundation](https://www.cncf.io/). So, I set out to learn more about Kubernetes.  

<!-- <div class="separator" style="clear: both; text-align: center;"> -->
![Alt Text]({static}/images/kubernetes.png "Kubernetes")
  
I started out by following the excellent [Hello Minikube](https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/) tutorial provided in the kubernetes docs. It steps you through installing local kubernetes (a.k.a. minikube), creating a docker image, deploying it to kubernetes, making it accessible outside the cluster, and live updating the running image. I followed the tutorial as written first, then applied it to my demo java project. Of course, I ran into some issues.

# Making Minikube Aware of your Docker Image
Minikube runs its own Docker daemon. As outlined [here](https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615), you have a few options for getting your docker images into minikube. Part of the hello minikube tutorial is to point your local docker client at the minikube docker daemon, and build your image there:

``` bash
$ eval $(minikube docker-env)$ env | grep DOCKER
DOCKER_HOST=tcp://192.168.64.2:2376
DOCKER_API_VERSION=1.23
DOCKER_TLS_VERIFY=1
DOCKER_CERT_PATH=/Users/ryanmckay/.minikube/certs
```

That works fine in the tutorial, because they are using the docker cli tool, which respects those env variables. Unfortunately, the [bmuschko gradle docker plugin](https://github.com/bmuschko/gradle-docker-plugin) does not. But it can be [configured](https://github.com/bmuschko/gradle-docker-plugin#extension-properties) to relatively easily. [java-docker-example v0.5.1](https://github.com/ryanmckaytx/java-docker-example/tree/v0.5.1) adds:


<script src="https://gist.github.com/ryanmckaytx/8dec1f69b1b1539f50b2c3a0c1dcad5e.js"></script>

So now you can build the docker image into kubernetes:

``` bash
$ ./gradlew buildImage
```

And you can stop pointing at kubernetes' docker instance with:  

``` bash
$ eval $(minikube docker-env -u)
```
I'm not sure this is the long term strategy for local dev, but at least it makes gradle and docker cli work the same way, which seems appropriate.

# Kubernetes Concepts

It's worth looking over the [kubernetes concepts docs](https://kubernetes.io/docs/concepts/) to understand the domain language.  A [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is a declaration of how you want your container deployed.  It specifies things like which image to deploy, how many instances it should have, ports to expose, etc.  A deployment is mutable. The configuration of a live deployment can be modified to, e.g. target a new docker image, change number of replicas, etc.  

A deployment manages one or more [replica sets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).  Each replica set corresponds to a distinct configuration of the deployment.  So if the docker image config is changed on the deployment, a new replica set representing the new config is created.  The deployment remembers the mapping from configuration to replica set, so if it sees the same configuration again, it will reuse an existing replica set. Replica sets managed by deployments should not be modified directly, even though the api allows it.

A replica set manages one or more [pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/), depending on the number of desired replicas.  In most cases, a pod runs a single container, though it can be configured to run [multiple containers](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#how-pods-manage-multiple-containers) that need to be colocated on the same cluster node.

# Create a Deployment

A complete deployment spec is a lengthy document, but kubernetes provides a quick and easy way to create one with minimal input:  

``` bash
$ kubectl run java-docker-example --image=ryanmckay/java-docker-example:0.0.1-SNAPSHOT --port=8080
deployment "java-docker-example” created
```

Then you can look at the deployment on the cli with:

``` bash
$ kubectl get deployment java-docker-example
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
java-docker-example   1         1         1            1           1d

$ kubectl describe deployment java-docker-example
Name:            java-docker-example
Namespace:        defaultCreationTimestamp:    Thu, 07 Sep 2017 00:00:37 -0500
Labels:            run=java-docker-example
Annotations:        deployment.kubernetes.io/revision=1
Selector:        run=java-docker-example
Replicas:        1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:        RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:    1 max unavailable, 1 max surge
Pod Template:  
    Labels:    run=java-docker-example  
    Containers:   java-docker-example:    
        Image:        ryanmckay/java-docker-example:0.0.1-SNAPSHOT    
        Port:        8080/TCP    
        Environment:    <none>    
        Mounts:        <none>  
    Volumes:        <none>
Conditions:  
    Type        Status    Reason  
    ----        ------    ------  
    Available     True    MinimumReplicasAvailable
OldReplicaSets:    <none>
NewReplicaSet:    java-docker-example-3948992014 (1/1 replicas created)
Events:  
    FirstSeen    LastSeen    Count    From            SubObjectPath    Type        Reason            Message  
    ---------    --------    -----    ----            -------------    --------    ------            -------  
    1d        1d        1    deployment-controller            Normal        ScalingReplicaSet    Scaled up replica set java-docker-example-3948992014 to 1
```

Notice the "Pod Template" section that describes the type of pod that will be managed by this deployment (through a replica set). At any given time, a deployment may be managing multiple active replica sets, which may in turn be managing multiple pods. In this example, there is only one replica set, and it is only managing one pod. But if you configured higher replication and rolling update, then during a change to the deployment spec, it will be managing spinning down the old replica set while spinning up the new replica set, at a minimum. If the spec changes faster than kubernetes can apply it, it could be more than that.  

The ownership relationship can be traversed at the command line. You can see the new and old replica set in the deployment description above. Replica set details can be obtained in similar fashion:  

``` bash
$ kubectl describe replicaset java-docker-example-3948992014
Name:  java-docker-example-3948992014
Namespace: default
Selector: pod-template-hash=3948992014,run=java-docker-example
Labels:  pod-template-hash=3948992014  run=java-docker-example
Annotations: deployment.kubernetes.io/desired-replicas=1  deployment.kubernetes.io/max-replicas=2  deployment.kubernetes.io/revision=1
Controlled By: Deployment/java-docker-example
Replicas: 1 current / 1 desired
Pods Status: 1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:  
    Labels: pod-template-hash=3948992014  run=java-docker-example  
    Containers:   
        java-docker-example:    
            Image:  ryanmckay/java-docker-example:0.0.1-SNAPSHOT    
            Port:  8080/TCP    
            Environment: <none>    
            Mounts:  <none>  
    Volumes:  <none>
Events:  FirstSeen LastSeen Count From   SubObjectPath Type  Reason   Message  
    --------- -------- ----- ----   ------------- -------- ------   -------  
    24m  24m  1 replicaset-controller   Normal  SuccessfulCreate Created pod: java-docker-example-3948992014-h1c0l
```

You can see the created pods in the replica set's Events log. It is worth noting that the "kubectl describe" command output is intended for human consumption. To get details in a machine readable format, use "kubectl get -o json".

# Minikube Dashboard

Its good to know the cli, but there is also the very nice

``` bash
$ minikube dashboard
```

That will launch your browser pointed at the minikube dashboard app. The information we saw at the cli is available and hyperlinked.

[![Dashboard Deployment]({static}/images/minikube-dashboard-deployment-thumb.jpg "Dashboard Deployment")]({static}/images/minikube-dashboard-deployment.png)
[![Dashboard Replicaset]({static}/images/minikube-dashboard-replicaset-thumb.jpg "Dashboard Replicaset")]({static}/images/minikube-dashboard-replicaset.png)
[![Dashboard Pod]({static}/images/minikube-dashboard-pod-thumb.jpg "Dashboard Pod")]({static}/images/minikube-dashboard-pod.png)

# Internal Access to Container {: style="clear: both"}
At this point, the deployed container is running, and you can see logs with:

``` bash
$ kubectl logs deployment/java-docker-example
$ kubectl logs java-docker-example-3948992014-h1c0l
```

You can access it from within the cluster. Note the pod's IP address from the Pod image above. The following will start another pod running busybox.  

``` bash
$ kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
# telnet 172.17.0.4:8080
GET /greeting
{"id":4,"content":"Hello, World!"}
```

There are some issues here. We had to know the IP address of the pod. Also, if we were running more replicas, we wouldn't want to be reaching out to one specific instance.  The way to expose pods in kubernetes is through a [service](https://kubernetes.io/docs/concepts/services-networking/service/). First, note the busybox pod's environment:  

``` bash
# env | sort
HOME=/root
HOSTNAME=busybox
KUBERNETES_PORT=tcp://10.0.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.0.0.1:443
KUBERNETES_PORT_443_TCP_ADDR=10.0.0.1
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
SHLVL=1
TERM=xterm
```

Now we launch a service for our deployment:

``` bash
$ kubectl expose deployment java-docker-exampleservice "java-docker-example" exposed

$ kubectl describe service java-docker-example
Name:   java-docker-example
Namespace:  default
Labels:   run=java-docker-example
Annotations:  <none>
Selector:  run=java-docker-example
Type:   ClusterIP
IP:   10.0.0.32
Port:   <unset> 8080/TCP
Endpoints:  172.17.0.4:8080
Session Affinity: None
Events:   <none>
```

Now if we restart our busybox pod, we will have some new env variables related to the new service.

``` bash
# env | sort
HOME=/root
HOSTNAME=busybox
JAVA_DOCKER_EXAMPLE_PORT=tcp://10.0.0.32:8080
JAVA_DOCKER_EXAMPLE_PORT_8080_TCP=tcp://10.0.0.32:8080
JAVA_DOCKER_EXAMPLE_PORT_8080_TCP_ADDR=10.0.0.32
JAVA_DOCKER_EXAMPLE_PORT_8080_TCP_PORT=8080
JAVA_DOCKER_EXAMPLE_PORT_8080_TCP_PROTO=tcp
JAVA_DOCKER_EXAMPLE_SERVICE_HOST=10.0.0.32
JAVA_DOCKER_EXAMPLE_SERVICE_PORT=8080
KUBERNETES_PORT=tcp://10.0.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.0.0.1:443
KUBERNETES_PORT_443_TCP_ADDR=10.0.0.1
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
SHLVL=1
TERM=xterm

# telnet $JAVA_DOCKER_EXAMPLE_SERVICE_HOST:$JAVA_DOCKER_EXAMPLE_SERVICE_PORT
GET /greeting
{"id":5,"content":"Hello, World!"}
```

There are a couple of points to make here.  First, if you plan for pods within the cluster to use a service, and you want to use the env variables for discovery, the service needs to be created *before* those consuming pods. Second, there are several different service types.  Since we didn't specify a type, we got the default, ClusterIP. This exposes the service only within the cluster.  

# External Access to Container
At some point you're going to want to expose your containers outside the cluster.  The service types build on each other.  

## NodePort Service Type

NodePort exposes the service externally on each node's IP at a static port.  This supports managing your own load balancer in front of the nodes. Notice that it also set up a ClusterIP at 10.0.0.131.  

``` bash
$ kubectl expose deployment java-docker-example --type=NodePortservice "java-docker-example" exposed

$ kubectl describe service java-docker-example
Name:   java-docker-example
Namespace:  default
Labels:   run=java-docker-example
Annotations:  <none>
Selector:  run=java-docker-example
Type:   NodePort
IP:   10.0.0.131
Port:   <unset> 8080/TCP
NodePort:  <unset> 32478/TCP
Endpoints:  172.17.0.4:8080
Session Affinity: None
Events:   <none>

$ kubectl get node minikube -o jsonpath='{.status.addresses[].address}'
192.168.99.100

$ curl 192.168.99.100:32478/greeting
{"id":6,"content":"Hello, World!"}
```

## LoadBalancer Service Type

This type will configure a cloud-based load balancer for you.  I need to learn more about this, as I did all these exercises on minikube only. Even on minikube though, LoadBalancer type makes your life easier.

``` bash
$ kubectl expose deployment java-docker-example --type=LoadBalancerservice "java-docker-example" exposed

$ kubectl describe service java-docker-example
Name:   java-docker-example
Namespace:  default
Labels:   run=java-docker-example
Annotations:  <none>
Selector:  run=java-docker-example
Type:   LoadBalancer
IP:   10.0.0.193
Port:   <unset> 8080/TCP
NodePort:  <unset> 32535/TCP
Endpoints:  172.17.0.4:8080
Session Affinity: None
Events:   <none>

$ minikube service java-docker-exampleOpening kubernetes service default/java-docker-example in default browser...
```

This saves you from having to track down and piece together the node ip and port.