Title: Docker Java Example Part 4: Bmuschko and Nebula Gradle Docker Plugins
Date: 2017-09-01 13:31
Author: Ryan McKay
Tags: gradle, docker
Slug: docker-java-example-part-4-bmuschko-nebula-gradle-docker-plugins
Status: draft

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

Converting from the [transmode](https://github.com/Transmode/gradle-docker) gradle plugin to the [bmuschko remote api gradle plugin](https://github.com/bmuschko/gradle-docker-plugin#remote-api-plugin) was pretty straightforward. Other than importing and applying the plugin, the code to get local docker image creation working is as follows:

</div>

</p>

<p>

<script src="https://gist.github.com/ryanmckaytx/79440cb30c30e2040800b31af51e0c34.js"></script>

</p>

  

<div>

</p>

Note that bmuschko does support multiple image tags, and I took advantage of that to get the versioned tag as well as the "latest" tag.

  

``` brush:
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZEryanmckay/java-docker-example   0.0.1-SNAPSHOT      7fd01d5b247f        6 seconds ago       115MBryanmckay/java-docker-example   latest              7fd01d5b247f        6 seconds ago       115MB
```

</p>

I tagged the code repo at this point [v0.4.1](https://github.com/ryanmckaytx/java-docker-example/tree/v0.4.1)  

<h2>

</p>

Java Application plugin

</h2>

</p>

</div>

</p>

<div>

</p>

In addition to the low-level remote api plugin, bmuschko offers an opinionated [docker-java-application](https://github.com/bmuschko/gradle-docker-plugin#java-application-plugin) plugin based on the [application gradle plugin](http://www.gradle.org/docs/current/userguide/application_plugin.html). Using the opinionated plugin cuts down dramatically on the boilerplate in the build.gradle:

</div>

</p>

  

<p>

<script src="https://gist.github.com/ryanmckaytx/d287ac6e8131aa1355629910fc19fd4e.js"></script>

</p>

  

<div>

</p>

Unfortunately, this task only supports one tag. By default, you get the versioned one.

  

``` brush:
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZEryanmckay/java-docker-example   0.0.1-snapshot      415a9e4b201d        3 seconds ago       115MB
```

</p>

</div>

</p>

<div>

</p>

The generated Dockerfile looks like this:  

  

<p>

<script src="https://gist.github.com/ryanmckaytx/a0b7ba41cdca53869a71c2f8c0e713e4.js"></script>

</p>

As an interesting side note, the [ADD](https://docs.docker.com/engine/reference/builder/#add) Dockerfile directive has special behavior when the file being added is a tar file. In that case, it unpacks it to the destination.  

  

</div>

</p>

<div>

</p>

The application gradle plugin is a more generic method of packaging up a java application than that offered by the spring boot plugin. It creates a tar file containing the application jar and all the dependency jars. It also contains a shell script for launching the application, which has OS detection and some OS-specific config.

</div>

</p>

<div>

</p>

``` brush:
$ tar tf build/distributions/java-docker-example-0.0.1-SNAPSHOT.tar java-docker-example-0.0.1-SNAPSHOT/java-docker-example-0.0.1-SNAPSHOT/lib/java-docker-example-0.0.1-SNAPSHOT/lib/java-docker-example-0.0.1-SNAPSHOT.jarjava-docker-example-0.0.1-SNAPSHOT/lib/spring-boot-starter-1.5.4.RELEASE.jarjava-docker-example-0.0.1-SNAPSHOT/lib/spring-boot-starter-web-1.5.4.RELEASE.jar...java-docker-example-0.0.1-SNAPSHOT/bin/java-docker-example-0.0.1-SNAPSHOT/bin/java-docker-examplejava-docker-example-0.0.1-SNAPSHOT/bin/java-docker-example.bat
```

</p>

  

</div>

</p>

<div>

</p>

I started using gradle about the same time I started using spring boot (which has its own [gradle plugin](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-gradle-plugin.html) with [executable jar packaging](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html)), so wasn't familiar with the application plugin. It makes sense that bmuschko would base the opinionated plugin on that, so it can support all types of java applications, not just spring boot.  However, since I plan to exclusively use spring boot for the foreseeable future, and can completely specify the execution environment in Docker (so don't need the OS-related functionality provided by the application plugin), I want to stick with Spring Boot application packaging and running.  

I left the modifications in a branch tagged as [v0.4.2](https://github.com/ryanmckaytx/java-docker-example/tree/v0.4.2)  

<h2>

</p>

Nebula docker gradle plugin

</h2>

</p>

</div>

</p>

<div>

</p>

Netflix publishes a set of plugins for gradle called [Nebula](https://github.com/nebula-plugins). The [nebula-docker-plugin](https://github.com/nebula-plugins/nebula-docker-plugin) is another opinionated plugin built on top of the bmuschko and application plugins.  It doesn't seem to add a lot beyond the bmuschko application plugin, other than the concept of separate test and production repositories for publishing docker images.  I'm going to look into docker deployment models next so it might come into play there.

</div>

</p>
