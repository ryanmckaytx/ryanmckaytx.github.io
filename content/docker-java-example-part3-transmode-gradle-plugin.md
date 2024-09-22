Title: Docker Java Example Part 3: Transmode Gradle plugin
Date: 2017-08-29 15:23
Author: Ryan McKay
Tags: gradle, docker
Slug: docker-java-example-part3-transmode-gradle-plugin
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

At last we get to some Docker in this Docker example.  

<a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjkS1UxytEi1RFoarWUj9WASlObDgxPUzxff1IA6GtmZYNqRgEK27Lv5GwfnRAuSRZ2It4A41u0XB6IHrGth2oWv3e2i1d1eypyxjkle_DiHb8CoS1PN4dtj8yWfQxZJFhsvELEUmP19ug/s1600/docker+logo.png" data-imageanchor="1"><img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjkS1UxytEi1RFoarWUj9WASlObDgxPUzxff1IA6GtmZYNqRgEK27Lv5GwfnRAuSRZ2It4A41u0XB6IHrGth2oWv3e2i1d1eypyxjkle_DiHb8CoS1PN4dtj8yWfQxZJFhsvELEUmP19ug/s200/docker+logo.png" data-border="0" data-original-height="250" data-original-width="297" width="200" height="168" /></a>

  

<h2>

</p>

Gradle Docker Plugin

</h2>

</p>

There are a few prominent docker plugins for gradle: [transmode](https://github.com/Transmode/gradle-docker), [bmuschko](https://github.com/bmuschko/gradle-docker-plugin), and Netflix [nebula](https://github.com/nebula-plugins/nebula-docker-plugin). First, I used transmode, as recommended in the spring boot guide [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/). After adding the Dockerfile:  

<div class="separator" style="clear: both; text-align: center;">

</p>

  

</div>

</p>

``` brush:
FROM openjdk:8-jdk-alpineVOLUME /tmpADD target/java-docker-example-0.0.1-SNAPSHOT.jar app.jarENV JAVA_OPTS=""ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
```

</p>

  

, and updating build.gradle as described in the guide, I was able to build a docker image for my application:  

  

``` brush:
$ gw clean build buildDocker --infoSetting up staging directory.Creating Dockerfile from file /Users/ryanmckay/projects/java-docker-example/java-docker-example/src/main/docker/Dockerfile.Determining image tag: ryanmckay/java-docker-example:0.0.1-SNAPSHOTUsing the native docker binary.Sending build context to Docker daemon  14.43MBStep 1/5 : FROM openjdk:8-jdk-alpine---> 478bf389b75bStep 2/5 : VOLUME /tmp---> Using cache---> 136f2d4e58dcStep 3/5 : ADD target/java-docker-example-0.0.1-SNAPSHOT.jar app.jar---> b3b47b89bbf1Removing intermediate container 92f637bc67e0Step 4/5 : ENV JAVA_OPTS ""---> Running in e90c9a3557eb---> 1d3f6526e8e5Removing intermediate container e90c9a3557ebStep 5/5 : ENTRYPOINT sh -c java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar---> Running in 2fbfb52f836d---> f001bdddc80bRemoving intermediate container 2fbfb52f836dSuccessfully built f001bdddc80bSuccessfully tagged ryanmckay/java-docker-example:0.0.1-SNAPSHOT$ docker imagesREPOSITORY                       TAG               IMAGE ID        CREATED          SIZEryanmckay/java-docker-example    0.0.1-SNAPSHOT    f001bdddc80b    3 minutes ago    115MB$ docker run -p 8080:8080 -t ryanmckay/java-docker-example:0.0.1-SNAPSHOT 
```

</p>

<h2>

</p>

Automatically Tracking Application Version

</h2>

</p>

<div>

</p>

Note that the [Dockerfile](https://github.com/ryanmckaytx/java-docker-example/blob/991edc8a2d668e3418aae9063b362b06315f6305/src/main/docker/Dockerfile) at this point has the application version hard-coded in it. This duplication must not stand. The transmode gradle plugin also supports a dsl for specifying the Dockerfile in build.gradle. Then as part of the build process, it produces the actual Dockerfile.  

  

I set about moving line by line of the Dockerfile into the dsl. With one exception it went smoothly. You can see the result in [v0.3](https://github.com/ryanmckaytx/java-docker-example/tree/v0.3) of the app. The relevant portion of build.gradle is listed here. You can see its pretty much a line for line translation of the Dockerfile. And since we have access to the jar filename in the build script, nothing needs to be hard coded for docker.

</div>

</p>

<div>

</p>

  

``` brush:
// for dockergroup = 'ryanmckay'docker { baseImage 'openjdk:8-jdk-alpine'}task buildDocker(type: Docker, dependsOn: build) { applicationName = jar.baseName volume('/tmp') addFile(jar.archivePath, 'app.jar') setEnvironment('JAVA_OPTS', '""') entryPoint([ 'sh', '-c', 'java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar' ])}
```

</p>

<h2>

</p>

ENV is used at build time And at run time

</h2>

</p>

</div>

</p>

<div>

</p>

One little gotcha in the previous section lead to an interesting learning. My initial attempt to set the JAVA_OPTS env variable looked like this:  

``` brush:
setEnvironment('JAVA_OPTS', '')
```

</p>

but that produced an illegal line in the Dockerfile:

  

``` brush:
ENV JAVA_OPTS
```

</p>

That led me to read about the [ENV](https://docs.docker.com/engine/reference/builder/#env) directive in the Dockerfile reference docs.  I was confused about whether ENV directives are used at build time, run time, or both.  Turns out, the answer is both, as I was able to prove to myself with the following. The ENV THE_FILE is used at build time to decide which file to add to the image, and at run time as an environment variable, which can be overriden at the command line.

  

  

``` brush:
$ cat somefilesomefile contents$ cat DockerfileFROM alpine:latestENV THE_FILE="somefile"ADD $THE_FILE containerizedfileENTRYPOINT ["sh", "-c", "cat containerizedfile && echo '-----' && env | sort"]$ docker build -t envtest .Sending build context to Docker daemon  3.072kBStep 1/4 : FROM alpine:latest ---> 7328f6f8b418Step 2/4 : ENV THE_FILE "somefile" ---> Using cache ---> 148a4236ce19Step 3/4 : ADD $THE_FILE containerizedfile ---> Using cache ---> d44f9e242685Step 4/4 : ENTRYPOINT sh -c cat containerizedfile && echo '-----' && env | sort ---> Running in 70de8ceac5ef ---> 5d875712904aRemoving intermediate container 70de8ceac5efSuccessfully built 5d875712904aSuccessfully tagged envtest:latest$ docker run envtestsomefile contents-----HOME=/rootHOSTNAME=36b3233697dfPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPWD=/SHLVL=1THE_FILE=somefileno_proxy=*.local, 169.254/16$ docker run -e THE_FILE=blah envtestsomefile contents-----HOME=/rootHOSTNAME=6a0f8c183a18PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPWD=/SHLVL=1THE_FILE=blahno_proxy=*.local, 169.254/16
```

</p>

</div>

</p>

<div>

</p>

<h2>

</p>

Version Tag and Latest Tag

</h2>

</p>

</div>

</p>

<div>

</p>

So that all worked fine for creating and publishing a versioned docker image locally.  But I really want that image tagged with the semantic version and also the **latest** tag.  The transmode plugin currently does not support multiple tags for the produced docker image.  I'm not the only one who [wants this feature](https://github.com/Transmode/gradle-docker/issues/98).  I took a look at the source code, and it wouldn't be a minor change.  At this point, I'm only publishing locally, so given the choice between version tag and latest tag, I'm going to go for latest for now.  This is a simple matter of [adding](https://github.com/ryanmckaytx/java-docker-example/commit/4f62d9aa5a5a3cf51160dd0cb230c19896bc6da2?diff=unified) tagVersion = 'latest' to the buildDocker task.  

  

I tagged the [code repo at v0.3.2](https://github.com/ryanmckaytx/java-docker-example/tree/v0.3.2) at this point.  

  

I'm going to move on to evaluating the [bmuschko](https://github.com/bmuschko/gradle-docker-plugin) and Netflix [Nebula Docker](https://github.com/nebula-plugins/nebula-docker-plugin) Gradle plugins next.

</div>

</p>
