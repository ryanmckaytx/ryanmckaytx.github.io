Title: Docker Java Example Part 5: Kubernetes
Date: 2017-09-14 17:28
Author: Ryan McKay
Tags: docker, kubernetes
Slug: docker-java-example-part-5-kubernetes
Status: published

<div class="toc">

</p>

<h3 style="padding-left: 2em;">

</p>

Docker Java Example Series

</h3>

</p>

<ol>

</p>

<p>

<li>

[Initializing a new Spring Boot Project](http://againstentropy.blogspot.com/2017/07/docker-java-example-part-1-initializing.html)
</li>

</p>

<p>

<li>

[Spring Web MVC Testing](http://againstentropy.blogspot.com/2017/08/docker-java-example-part-2-spring-web.html)
</li>

</p>

<p>

<li>

[Transmode Gradle Plugin](http://againstentropy.blogspot.com/2017/08/docker-java-example-part3-transmode-gradle-plugin.html)
</li>

</p>

<p>

<li>

[Bmuschko and Nebula Gradle Plugins](http://againstentropy.blogspot.com/2017/09/docker-java-example-part-4-bmuschko-nebula-gradle-docker-plugins.html)
</li>

</p>

<p>

<li>

[Kubernetes](https://againstentropy.blogspot.com/2017/09/docker-java-example-part-5-kubernetes.html)
</li>

</p>

</ol>

</p>

</div>

</p>

<div>

</p>

Now that I've got my project getting packaged up in a docker image, the next step in this POC is to look at platforms for running docker. The only PaaS that I am familiar with now is Pivotal Cloud Foundry, which we used at my last job to deploy Spring Boot executable jars. PCF was working on a docker story, not sure how far that got. It looks like they are pretty [bought into Kubernetes](https://pivotal.io/pks) these days. In fact it seems like the whole cloud world is moving in that direction, with the likes of Pivotal, VMware, Amazon, Microsoft, Dell, Alibaba, and Mesosphere joining the [Cloud Native Computing Foundation](https://www.cncf.io/). So, I set out to learn more about Kubernetes.  

<div class="separator" style="clear: both; text-align: center;">

</p>

<a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiHLmP57P2RU4Gvc4s71vN4ySFsuGhtPXwozaBHsHiCpCuK5Xg98AJu5xVqUtuNGienRskIQlm1ivfkLUERHoM6jjFMudLi3raFTRMXE40RWlFd0iQ4atNm-AGzAVgwCKGyMWG4a4Ma36I/s1600/kubernetes.png" data-imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiHLmP57P2RU4Gvc4s71vN4ySFsuGhtPXwozaBHsHiCpCuK5Xg98AJu5xVqUtuNGienRskIQlm1ivfkLUERHoM6jjFMudLi3raFTRMXE40RWlFd0iQ4atNm-AGzAVgwCKGyMWG4a4Ma36I/s1600/kubernetes.png" data-border="0" data-original-height="240" data-original-width="280" /></a>

</div>

</p>

</div>

</p>

<div>

</p>

  

I started out by following the excellent [Hello Minikube](https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/) tutorial provided in the kubernetes docs. It steps you through installing local kubernetes (a.k.a. minikube), creating a docker image, deploying it to kubernetes, making it accessible outside the cluster, and live updating the running image. I followed the tutorial as written first, then applied it to my demo java project. Of course, I ran into some issues.

</div>

</p>

<h2>

</p>

Making Minikube Aware of your Docker Image

</h2>

</p>

<div>

</p>

Minikube runs its own Docker daemon. As outlined [here](https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615), you have a few options for getting your docker images into minikube. Part of the hello minikube tutorial is to point your local docker client at the minikube docker daemon, and build your image there:

  

``` brush:
$ eval $(minikube docker-env)$ env | grep DOCKERDOCKER_HOST=tcp://192.168.64.2:2376DOCKER_API_VERSION=1.23DOCKER_TLS_VERIFY=1DOCKER_CERT_PATH=/Users/ryanmckay/.minikube/certs
```

</p>

That works fine in the tutorial, because they are using the docker cli tool, which respects those env variables. Unfortunately, the [bmuschko gradle docker plugin](https://github.com/bmuschko/gradle-docker-plugin) does not. But it can be [configured](https://github.com/bmuschko/gradle-docker-plugin#extension-properties) to relatively easily. [java-docker-example v0.5.1](https://github.com/ryanmckaytx/java-docker-example/tree/v0.5.1) adds:

<p>

<script src="https://gist.github.com/ryanmckaytx/8dec1f69b1b1539f50b2c3a0c1dcad5e.js"></script>

</p>

  

So now you can build the docker image into kubernetes:

  

``` brush:
$ ./gradlew buildImage
```

</p>

And you can stop pointing at kubernetes' docker instance with:  

``` brush:
$ eval $(minikube docker-env -u)
```

</p>

I'm not sure this is the long term strategy for local dev, but at least it makes gradle and docker cli work the same way, which seems appropriate.

</div>

</p>

<div>

</p>

<h2>

</p>

Kubernetes Concepts

</h2>

</p>

</div>

</p>

<div>

</p>

It's worth looking over the [kubernetes concepts docs](https://kubernetes.io/docs/concepts/) to understand the domain language.  A [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is a declaration of how you want your container deployed.  It specifies things like which image to deploy, how many instances it should have, ports to expose, etc.  A deployment is mutable. The configuration of a live deployment can be modified to, e.g. target a new docker image, change number of replicas, etc.  

</div>

</p>

<div>

</p>

  

</div>

</p>

<div>

</p>

A deployment manages one or more [replica sets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).  Each replica set corresponds to a distinct configuration of the deployment.  So if the docker image config is changed on the deployment, a new replica set representing the new config is created.  The deployment remembers the mapping from configuration to replica set, so if it sees the same configuration again, it will reuse an existing replica set. Replica sets managed by deployments should not be modified directly, even though the api allows it.

</div>

</p>

<div>

</p>

  

</div>

</p>

<div>

</p>

A replica set manages one or more [pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/), depending on the number of desired replicas.  In most cases, a pod runs a single container, though it can be configured to run [multiple containers](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#how-pods-manage-multiple-containers) that need to be colocated on the same cluster node.

</div>

</p>

<h2>

</p>

Create a Deployment

</h2>

</p>

<div>

</p>

A complete deployment spec is a lengthy document, but kubernetes provides a quick and easy way to create one with minimal input:  

``` brush:
$ kubectl run java-docker-example --image=ryanmckay/java-docker-example:0.0.1-SNAPSHOT --port=8080deployment "java-docker-example” created
```

</p>

Then you can look at the deployment on the cli with:

  

``` brush:
$ kubectl get deployment java-docker-exampleNAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGEjava-docker-example   1         1         1            1           1d$ kubectl describe deployment java-docker-exampleName:            java-docker-exampleNamespace:        defaultCreationTimestamp:    Thu, 07 Sep 2017 00:00:37 -0500Labels:            run=java-docker-exampleAnnotations:        deployment.kubernetes.io/revision=1Selector:        run=java-docker-exampleReplicas:        1 desired | 1 updated | 1 total | 1 available | 0 unavailableStrategyType:        RollingUpdateMinReadySeconds:    0RollingUpdateStrategy:    1 max unavailable, 1 max surgePod Template<:  Labels:    run=java-docker-example  Containers:   java-docker-example:    Image:        ryanmckay/java-docker-example:0.0.1-SNAPSHOT    Port:        8080/TCP    Environment:    <none>    Mounts:        <none>  Volumes:        <none>Conditions:  Type        Status    Reason  ----        ------    ------  Available     True    MinimumReplicasAvailableOldReplicaSets:    <none>NewReplicaSet:    java-docker-example-3948992014 (1/1 replicas created)Events:  FirstSeen    LastSeen    Count    From            SubObjectPath    Type        Reason            Message  ---------    --------    -----    ----            -------------    --------    ------            -------  1d        1d        1    deployment-controller            Normal        ScalingReplicaSet    Scaled up replica set java-docker-example-3948992014 to 1
```

</p>

</div>

</p>

Notice the "Pod Template" section that describes the type of pod that will be managed by this deployment (through a replica set). At any given time, a deployment may be managing multiple active replica sets, which may in turn be managing multiple pods. In this example, there is only one replica set, and it is only managing one pod. But if you configured higher replication and rolling update, then during a change to the deployment spec, it will be managing spinning down the old replica set while spinning up the new replica set, at a minimum. If the spec changes faster than kubernetes can apply it, it could be more than that.  

  

The ownership relationship can be traversed at the command line. You can see the new and old replica set in the deployment description above. Replica set details can be obtained in similar fashion:  

``` brush:
$ kubectl describe replicaset java-docker-example-3948992014Name:  java-docker-example-3948992014Namespace: defaultSelector: pod-template-hash=3948992014,run=java-docker-exampleLabels:  pod-template-hash=3948992014  run=java-docker-exampleAnnotations: deployment.kubernetes.io/desired-replicas=1  deployment.kubernetes.io/max-replicas=2  deployment.kubernetes.io/revision=1Controlled By: Deployment/java-docker-exampleReplicas: 1 current / 1 desiredPods Status: 1 Running / 0 Waiting / 0 Succeeded / 0 FailedPod Template:  Labels: pod-template-hash=3948992014  run=java-docker-example  Containers:   java-docker-example:    Image:  ryanmckay/java-docker-example:0.0.1-SNAPSHOT    Port:  8080/TCP    Environment: <none>    Mounts:  <none>  Volumes:  <none>Events:  FirstSeen LastSeen Count From   SubObjectPath Type  Reason   Message  --------- -------- ----- ----   ------------- -------- ------   -------  24m  24m  1 replicaset-controller   Normal  SuccessfulCreate Created pod: java-docker-example-3948992014-h1c0l
```

</p>

You can see the created pods in the replica set's Events log. It is worth noting that the "kubectl describe" command output is intended for human consumption. To get details in a machine readable format, use "kubectl get -o json".

  

<h2>

</p>

Minikube Dashboard

</h2>

</p>

<div>

</p>

Its good to know the cli, but there is also the very nice

  

``` brush:
$ minikube dashboard
```

</p>

  

That will launch your browser pointed at the minikube dashboard app. The information we saw at the cli is available and hyperlinked.

  

  

<div class="separator" style="clear: both; text-align: center;">

</p>

<a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEihxd1Q5AAwUYjTmQdpy4hDdrga2Cfr6aKy-7VeUQqan5SuMf9NLOiKnmmg9LIUVQ1fb-GcZ2aLyLsN0EUhPsRNmG5xTwhlhBHEKWus1GMxAL52zKLhyphenhyphen1CDiKYpxJH8C5hMIdK8WqVWq5M/s1600/minikube-dashboard-deployment.png" data-imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"><img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEihxd1Q5AAwUYjTmQdpy4hDdrga2Cfr6aKy-7VeUQqan5SuMf9NLOiKnmmg9LIUVQ1fb-GcZ2aLyLsN0EUhPsRNmG5xTwhlhBHEKWus1GMxAL52zKLhyphenhyphen1CDiKYpxJH8C5hMIdK8WqVWq5M/s200/minikube-dashboard-deployment.png" data-border="0" data-original-height="1092" data-original-width="1600" height="100" /></a>

<a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg9-jJwzONXnIK3rVD8_ciUEOmIh042cE8Tq8PZFGrBQtzDGAVrO9nVocqkfhSPL6pMVotlFsSlfDtqGGbMrLWpmi4gdkbth1GBCrhBZmL-GNzsy36GxnyZdsKpcCF4SPm8EY0i7I9wNbI/s1600/minikube-dashboard-replicaset.png" data-imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg9-jJwzONXnIK3rVD8_ciUEOmIh042cE8Tq8PZFGrBQtzDGAVrO9nVocqkfhSPL6pMVotlFsSlfDtqGGbMrLWpmi4gdkbth1GBCrhBZmL-GNzsy36GxnyZdsKpcCF4SPm8EY0i7I9wNbI/s200/minikube-dashboard-replicaset.png" data-border="0" data-original-height="1220" data-original-width="1600" height="100" /></a>

<a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg5TLAxdy8kDiVVXlwVvzStzCjrWEk7XlutSBH3a7FS5hDjVe6MZ9AL_fgn2eFvX7rmgJGbb0Nh4j93uaZlQR2GEB_A0Cu-v5uxJT6kIe3SXy3Jcsp26qXRotqdiQTMBFuPPIVzFdcK7A4/s1600/minikube-dashboard-pod.png" data-imageanchor="1" style="clear: right; float: right; margin-bottom: 1em; margin-left: 1em;"><img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg5TLAxdy8kDiVVXlwVvzStzCjrWEk7XlutSBH3a7FS5hDjVe6MZ9AL_fgn2eFvX7rmgJGbb0Nh4j93uaZlQR2GEB_A0Cu-v5uxJT6kIe3SXy3Jcsp26qXRotqdiQTMBFuPPIVzFdcK7A4/s200/minikube-dashboard-pod.png" data-border="0" data-original-height="966" data-original-width="1600" height="100" /></a>

</div>

</p>

  

<h2>

</p>

Internal Access to Container

</h2>

</p>

At this point, the deployed container is running, and you can see logs with:

  

``` brush:
$ kubectl logs deployment/java-docker-example$ kubectl logs java-docker-example-3948992014-h1c0l
```

</p>

You can access it from within the cluster. Note the pod's IP address from the Pod image above. The following will start another pod running busybox.  

``` brush:
$ kubectl run -i --tty busybox --image=busybox --restart=Never -- sh/ # telnet 172.17.0.4:8080GET /greeting{"id":4,"content":"Hello, World!"}
```

</p>

There are some issues here. We had to know the IP address of the pod. Also, if we were running more replicas, we wouldn't want to be reaching out to one specific instance.  The way to expose pods in kubernetes is through a [service](https://kubernetes.io/docs/concepts/services-networking/service/). First, note the busybox pod's environment:  

  

``` brush:
/ # env | sortHOME=/rootHOSTNAME=busyboxKUBERNETES_PORT=tcp://10.0.0.1:443KUBERNETES_PORT_443_TCP=tcp://10.0.0.1:443KUBERNETES_PORT_443_TCP_ADDR=10.0.0.1KUBERNETES_PORT_443_TCP_PORT=443KUBERNETES_PORT_443_TCP_PROTO=tcpKUBERNETES_SERVICE_HOST=10.0.0.1KUBERNETES_SERVICE_PORT=443KUBERNETES_SERVICE_PORT_HTTPS=443PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPWD=/SHLVL=1TERM=xterm
```

</p>

Now we launch a service for our deployment:

  

``` brush:
$ kubectl expose deployment java-docker-exampleservice "java-docker-example" exposed$ kubectl describe service java-docker-exampleName:   java-docker-exampleNamespace:  defaultLabels:   run=java-docker-exampleAnnotations:  <none>Selector:  run=java-docker-exampleType:   ClusterIPIP:   10.0.0.32Port:   <unset> 8080/TCPEndpoints:  172.17.0.4:8080Session Affinity: NoneEvents:   <none>
```

</p>

Now if we restart our busybox pod, we will have some new env variables related to the new service.

  

``` brush:
/ # env | sortHOME=/rootHOSTNAME=busyboxJAVA_DOCKER_EXAMPLE_PORT=tcp://10.0.0.32:8080JAVA_DOCKER_EXAMPLE_PORT_8080_TCP=tcp://10.0.0.32:8080JAVA_DOCKER_EXAMPLE_PORT_8080_TCP_ADDR=10.0.0.32JAVA_DOCKER_EXAMPLE_PORT_8080_TCP_PORT=8080JAVA_DOCKER_EXAMPLE_PORT_8080_TCP_PROTO=tcpJAVA_DOCKER_EXAMPLE_SERVICE_HOST=10.0.0.32JAVA_DOCKER_EXAMPLE_SERVICE_PORT=8080KUBERNETES_PORT=tcp://10.0.0.1:443KUBERNETES_PORT_443_TCP=tcp://10.0.0.1:443KUBERNETES_PORT_443_TCP_ADDR=10.0.0.1KUBERNETES_PORT_443_TCP_PORT=443KUBERNETES_PORT_443_TCP_PROTO=tcpKUBERNETES_SERVICE_HOST=10.0.0.1KUBERNETES_SERVICE_PORT=443KUBERNETES_SERVICE_PORT_HTTPS=443PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPWD=/SHLVL=1TERM=xterm/ # telnet $JAVA_DOCKER_EXAMPLE_SERVICE_HOST:$JAVA_DOCKER_EXAMPLE_SERVICE_PORTGET /greeting{"id":5,"content":"Hello, World!"}
```

</p>

There are a couple of points to make here.  First, if you plan for pods within the cluster to use a service, and you want to use the env variables for discovery, the service needs to be created *before* those consuming pods. Second, there are several different service types.  Since we didn't specify a type, we got the default, ClusterIP. This exposes the service only within the cluster.  

<h2>

</p>

External Access to Container

</h2>

</p>

At some point you're going to want to expose your containers outside the cluster.  The service types build on each other.  

  

<h3>

</p>

NodePort Service Type

</h3>

</p>

NodePort exposes the service externally on each node's IP at a static port.  This supports managing your own load balancer in front of the nodes. Notice that it also set up a ClusterIP at 10.0.0.131.  

  

``` brush:
$ kubectl expose deployment java-docker-example --type=NodePortservice "java-docker-example" exposed$ kubectl describe service java-docker-exampleName:   java-docker-exampleNamespace:  defaultLabels:   run=java-docker-exampleAnnotations:  <none>Selector:  run=java-docker-exampleType:   NodePortIP:   10.0.0.131Port:   <unset> 8080/TCPNodePort:  <unset> 32478/TCPEndpoints:  172.17.0.4:8080Session Affinity: NoneEvents:   <none>$ kubectl get node minikube -o jsonpath='{.status.addresses[].address}'192.168.99.100$ curl 192.168.99.100:32478/greeting{"id":6,"content":"Hello, World!"}
```

</p>

<h3>

</p>

LoadBalancer Service Type

</h3>

</p>

<div>

</p>

This type will configure a cloud-based load balancer for you.  I need to learn more about this, as I did all these exercises on minikube only. Even on minikube though, LoadBalancer type makes your life easier.

</div>

</p>

``` brush:
$ kubectl expose deployment java-docker-example --type=LoadBalancerservice "java-docker-example" exposed$ kubectl describe service java-docker-exampleName:   java-docker-exampleNamespace:  defaultLabels:   run=java-docker-exampleAnnotations:  <none>Selector:  run=java-docker-exampleType:   LoadBalancerIP:   10.0.0.193Port:   <unset> 8080/TCPNodePort:  <unset> 32535/TCPEndpoints:  172.17.0.4:8080Session Affinity: NoneEvents:   <none>$ minikube service java-docker-exampleOpening kubernetes service default/java-docker-example in default browser...
```

</p>

This saves you from having to track down and piece together the node ip and port.

</div>

</p>
