Title: Docker Java Example Part 4: Bmuschko and Nebula Gradle Docker Plugins
Date: 2017-09-01 13:31
Author: Ryan McKay
Tags: gradle, docker
Slug: docker-java-example-part-4-bmuschko-nebula-gradle-docker-plugins
Status: published
Summary: Converting from the [transmode](https://github.com/Transmode/gradle-docker) gradle plugin to the [bmuschko remote api gradle plugin](https://github.com/bmuschko/gradle-docker-plugin#remote-api-plugin) was pretty straightforward. Other than importing and applying the plugin, the code to get local docker image creation working is as follows

<div class="toc" markdown="1">
<div class="toctitle">Docker Java Example Series</div>

1. [Initializing a new Spring Boot Project](/docker-java-example-part-1-initializing.html)
2. [Spring Web MVC Testing](/docker-java-example-part-2-spring-web.html)
3. [Transmode Gradle Plugin](/docker-java-example-part3-transmode-gradle-plugin.html)
4. [Bmuschko and Nebula Gradle Plugins](/docker-java-example-part-4-bmuschko-nebula-gradle-docker-plugins.html)
5. [Kubernetes](/docker-java-example-part-5-kubernetes.html)
</div>

Converting from the [transmode](https://github.com/Transmode/gradle-docker) gradle plugin to the [bmuschko remote api gradle plugin](https://github.com/bmuschko/gradle-docker-plugin#remote-api-plugin) was pretty straightforward. Other than importing and applying the plugin, the code to get local docker image creation working is as follows:
<div style="clear: both; text-align: center;"> </div>

<script src="https://gist.github.com/ryanmckaytx/79440cb30c30e2040800b31af51e0c34.js"></script>

Note that bmuschko does support multiple image tags, and I took advantage of that to get the versioned tag as well as the "latest" tag.

``` bash
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
ryanmckay/java-docker-example   0.0.1-SNAPSHOT      7fd01d5b247f        6 seconds ago       115MB
ryanmckay/java-docker-example   latest              7fd01d5b247f        6 seconds ago       115MB
```

I tagged the code repo at this point [v0.4.1](https://github.com/ryanmckaytx/java-docker-example/tree/v0.4.1)  

## Java Application plugin
In addition to the low-level remote api plugin, bmuschko offers an opinionated [docker-java-application](https://github.com/bmuschko/gradle-docker-plugin#java-application-plugin) plugin based on the [application gradle plugin](http://www.gradle.org/docs/current/userguide/application_plugin.html). Using the opinionated plugin cuts down dramatically on the boilerplate in the build.gradle:

<script src="https://gist.github.com/ryanmckaytx/d287ac6e8131aa1355629910fc19fd4e.js"></script>

Unfortunately, this task only supports one tag. By default, you get the versioned one.

``` bash
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
ryanmckay/java-docker-example   0.0.1-snapshot      415a9e4b201d        3 seconds ago       115MB
```

The generated Dockerfile looks like this:  
<script src="https://gist.github.com/ryanmckaytx/a0b7ba41cdca53869a71c2f8c0e713e4.js"></script>

As an interesting side note, the [ADD](https://docs.docker.com/engine/reference/builder/#add) Dockerfile directive has special behavior when the file being added is a tar file. In that case, it unpacks it to the destination.  

The application gradle plugin is a more generic method of packaging up a java application than that offered by the spring boot plugin. It creates a tar file containing the application jar and all the dependency jars. It also contains a shell script for launching the application, which has OS detection and some OS-specific config.

``` bash
$ tar tf build/distributions/java-docker-example-0.0.1-SNAPSHOT.tar 
java-docker-example-0.0.1-SNAPSHOT/
java-docker-example-0.0.1-SNAPSHOT/lib/
java-docker-example-0.0.1-SNAPSHOT/lib/java-docker-example-0.0.1-SNAPSHOT.jar
java-docker-example-0.0.1-SNAPSHOT/lib/spring-boot-starter-1.5.4.RELEASE.jar
java-docker-example-0.0.1-SNAPSHOT/lib/spring-boot-starter-web-1.5.4.RELEASE.jar
...
java-docker-example-0.0.1-SNAPSHOT/bin/
java-docker-example-0.0.1-SNAPSHOT/bin/java-docker-example
java-docker-example-0.0.1-SNAPSHOT/bin/java-docker-example.bat
```

I started using gradle about the same time I started using spring boot (which has its own [gradle plugin](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-gradle-plugin.html) with [executable jar packaging](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html)), so wasn't familiar with the application plugin. It makes sense that bmuschko would base the opinionated plugin on that, so it can support all types of java applications, not just spring boot.  However, since I plan to exclusively use spring boot for the foreseeable future, and can completely specify the execution environment in Docker (so don't need the OS-related functionality provided by the application plugin), I want to stick with Spring Boot application packaging and running.  

I left the modifications in a branch tagged as [v0.4.2](https://github.com/ryanmckaytx/java-docker-example/tree/v0.4.2)  

## Nebula docker gradle plugin
Netflix publishes a set of plugins for gradle called [Nebula](https://github.com/nebula-plugins). The [nebula-docker-plugin](https://github.com/nebula-plugins/nebula-docker-plugin) is another opinionated plugin built on top of the bmuschko and application plugins.  It doesn't seem to add a lot beyond the bmuschko application plugin, other than the concept of separate test and production repositories for publishing docker images.  I'm going to look into docker deployment models next so it might come into play there.