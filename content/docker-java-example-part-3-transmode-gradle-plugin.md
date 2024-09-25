Title: Docker Java Example Part 3: Transmode Gradle plugin
Date: 2017-08-29 15:23
Author: Ryan McKay
Tags: gradle, docker
Slug: docker-java-example-part3-transmode-gradle-plugin
Status: published
Summary: At last we get to some Docker in this Docker example.  

<div class="toc" markdown="1">
<div class="toctitle">Docker Java Example Series</div>

1. [Initializing a new Spring Boot Project](/docker-java-example-part-1-initializing.html)
2. [Spring Web MVC Testing](/docker-java-example-part-2-spring-web.html)
3. [Transmode Gradle Plugin](/docker-java-example-part3-transmode-gradle-plugin.html)
4. [Bmuschko and Nebula Gradle Plugins](/docker-java-example-part-4-bmuschko-nebula-gradle-docker-plugins.html)
5. [Kubernetes](/docker-java-example-part-5-kubernetes.html)
</div>

At last we get to some Docker in this Docker example.  

![Docker logo]({static}/images/docker-logo.png "Docker")

## Gradle Docker Plugin
There are a few prominent docker plugins for gradle: [transmode](https://github.com/Transmode/gradle-docker), [bmuschko](https://github.com/bmuschko/gradle-docker-plugin), and Netflix [nebula](https://github.com/nebula-plugins/nebula-docker-plugin). First, I used transmode, as recommended in the spring boot guide [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/). After adding the Dockerfile:  

``` bash
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD target/java-docker-example-0.0.1-SNAPSHOT.jar app.jar
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
```

, and updating build.gradle as described in the guide, I was able to build a docker image for my application:  

``` bash
$ gw clean build buildDocker --info

Setting up staging directory.
Creating Dockerfile from file /Users/ryanmckay/projects/java-docker-example/java-docker-example/src/main/docker/Dockerfile.
Determining image tag: ryanmckay/java-docker-example:0.0.1-SNAPSHOT
Using the native docker binary.
Sending build context to Docker daemon  14.43MB
Step 1/5 : FROM openjdk:8-jdk-alpine
---> 478bf389b75b
Step 2/5 : VOLUME /tmp
---> Using cache
---> 136f2d4e58dc
Step 3/5 : ADD target/java-docker-example-0.0.1-SNAPSHOT.jar app.jar
---> b3b47b89bbf1
Removing intermediate container 92f637bc67e0
Step 4/5 : ENV JAVA_OPTS ""
---> Running in e90c9a3557eb
---> 1d3f6526e8e5
Removing intermediate container e90c9a3557eb
Step 5/5 : ENTRYPOINT sh -c java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar
---> Running in 2fbfb52f836d
---> f001bdddc80b
Removing intermediate container 2fbfb52f836d
Successfully built f001bdddc80b
Successfully tagged ryanmckay/java-docker-example:0.0.1-SNAPSHOT

$ docker images
REPOSITORY                       TAG               IMAGE ID        CREATED          SIZE
ryanmckay/java-docker-example    0.0.1-SNAPSHOT    f001bdddc80b    3 minutes ago    115MB

$ docker run -p 8080:8080 -t ryanmckay/java-docker-example:0.0.1-SNAPSHOT
```

## Automatically Tracking Application Version
Note that the [Dockerfile](https://github.com/ryanmckaytx/java-docker-example/blob/991edc8a2d668e3418aae9063b362b06315f6305/src/main/docker/Dockerfile) at this point has the application version hard-coded in it. This duplication must not stand. The transmode gradle plugin also supports a dsl for specifying the Dockerfile in build.gradle. Then as part of the build process, it produces the actual Dockerfile.  

I set about moving line by line of the Dockerfile into the dsl. With one exception it went smoothly. You can see the result in [v0.3](https://github.com/ryanmckaytx/java-docker-example/tree/v0.3) of the app. The relevant portion of build.gradle is listed here. You can see its pretty much a line for line translation of the Dockerfile. And since we have access to the jar filename in the build script, nothing needs to be hard coded for docker.

``` groovy
// for docker
group = 'ryanmckay'

docker {
 baseImage 'openjdk:8-jdk-alpine'
}

task buildDocker(type: Docker, dependsOn: build) {
 applicationName = jar.baseName
 volume('/tmp')
 addFile(jar.archivePath, 'app.jar')
 setEnvironment('JAVA_OPTS', '""')
 entryPoint([ 'sh', '-c', 'java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar' ])
}
```

## ENV is used at build time And at run time
One little gotcha in the previous section lead to an interesting learning. My initial attempt to set the JAVA_OPTS env variable looked like this:  

``` bash
setEnvironment('JAVA_OPTS', '')
```
but that produced an illegal line in the Dockerfile:

``` bash
ENV JAVA_OPTS
```

That led me to read about the [ENV](https://docs.docker.com/engine/reference/builder/#env) directive in the Dockerfile reference docs.  I was confused about whether ENV directives are used at build time, run time, or both.  Turns out, the answer is both, as I was able to prove to myself with the following. The ENV THE_FILE is used at build time to decide which file to add to the image, and at run time as an environment variable, which can be overriden at the command line.

``` bash
$ cat somefile
somefile contents

$ cat Dockerfile
FROM alpine:latest
ENV THE_FILE="somefile"
ADD $THE_FILE containerizedfile
ENTRYPOINT ["sh", "-c", "cat containerizedfile && echo '-----' && env | sort"]

$ docker build -t envtest .
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM alpine:latest
 ---> 7328f6f8b418
Step 2/4 : ENV THE_FILE "somefile"
 ---> Using cache
 ---> 148a4236ce19
Step 3/4 : ADD $THE_FILE containerizedfile
 ---> Using cache
 ---> d44f9e242685
Step 4/4 : ENTRYPOINT sh -c cat containerizedfile && echo '-----' && env | sort
 ---> Running in 70de8ceac5ef
 ---> 5d875712904a
Removing intermediate container 70de8ceac5ef
Successfully built 5d875712904a
Successfully tagged envtest:latest

$ docker run envtest
somefile contents
-----
HOME=/root
HOSTNAME=36b3233697df
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
SHLVL=1
THE_FILE=somefile
no_proxy=*.local, 169.254/16

$ docker run -e THE_FILE=blah envtest
somefile contents
-----
HOME=/root
HOSTNAME=6a0f8c183a18
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
SHLVL=1
THE_FILE=blah
no_proxy=*.local, 169.254/16
```

## Version Tag and Latest Tag
So that all worked fine for creating and publishing a versioned docker image locally.  But I really want that image tagged with the semantic version and also the **latest** tag.  The transmode plugin currently does not support multiple tags for the produced docker image.  I'm not the only one who [wants this feature](https://github.com/Transmode/gradle-docker/issues/98).  I took a look at the source code, and it wouldn't be a minor change.  At this point, I'm only publishing locally, so given the choice between version tag and latest tag, I'm going to go for latest for now.  This is a simple matter of [adding](https://github.com/ryanmckaytx/java-docker-example/commit/4f62d9aa5a5a3cf51160dd0cb230c19896bc6da2?diff=unified) tagVersion = 'latest' to the buildDocker task.  

I tagged the [code repo at v0.3.2](https://github.com/ryanmckaytx/java-docker-example/tree/v0.3.2) at this point.  

I'm going to move on to evaluating the [bmuschko](https://github.com/bmuschko/gradle-docker-plugin) and Netflix [Nebula Docker](https://github.com/nebula-plugins/nebula-docker-plugin) Gradle plugins next.