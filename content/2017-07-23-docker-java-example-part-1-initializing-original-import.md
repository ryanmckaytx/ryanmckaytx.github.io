Title: Docker Java Example Part 1: Initializing a new Spring Boot Project
Date: 2017-07-23 21:12
Author: Ryan McKay
Tags: gradle, spring
Slug: docker-java-example-part-1-initializing-original-import
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

I've been wanting to learn more about Docker for a while.  I'm almost done with this udemy course [Docker for Java Developers](https://www.udemy.com/docker-for-java-developers/learn/v4/overview).  Its a good course, and the concepts are straightforward.  To help me commit it to memory, I wanted to do my own project to apply what I'm learning.  

<https://github.com/ryanmckaytx/java-docker-example>  

In this part, I'm just going to initialize a new project.  [Part 2](http://againstentropy.blogspot.com/2017/08/docker-java-example-part-2-spring-web.html) covers Spring Web MVC testing.  

<h2>

</p>

Set up your dev machine

</h2>

</p>

<div>

</p>

Every once in a while, like when you switch jobs and get a new laptop, you need to set up a machine for java development.  

</div>

</p>

<div>

</p>

<div class="separator" style="clear: both; text-align: center;">

</p>

<a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhrLzE29MOewZBBM240Yz_MZiZ12uzWGYpq1K-t-D0aaOSekTtLO5fwJsgF8L5N9jKLLObd_wv2vNq2aDLUfaP9fZV59YFSE12aHsk5ZWg13GMp04DO-2M7sWtvVekDP5NwBi2ZXZbwXH4/s1600/sdk-man-small-pattern.png" data-imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhrLzE29MOewZBBM240Yz_MZiZ12uzWGYpq1K-t-D0aaOSekTtLO5fwJsgF8L5N9jKLLObd_wv2vNq2aDLUfaP9fZV59YFSE12aHsk5ZWg13GMp04DO-2M7sWtvVekDP5NwBi2ZXZbwXH4/s1600/sdk-man-small-pattern.png" data-border="0" data-original-height="117" data-original-width="200" /></a>

</div>

</p>

  

</div>

</p>

<div>

</p>

For this, I like to use [sdkman](http://sdkman.io/).  It helps install the tools you need to do java development.  It also helps switch between multiple versions of those tools.  I've installed java, groovy, gradle, and maven.

</div>

</p>

<div>

</p>

``` brush:
$ sdk cUsing:gradle: 4.0groovy: 2.4.11java: 8u131-zulumaven: 3.5.0
```

</p>

</div>

</p>

<h2>

</p>

Create a new project

</h2>

</p>

There are a few good ways to do this.  The typical way I do this is to copy another project.  Within an organization, or at least within a team, there is typically some amount of infrastructure and institutional knowledge built into existing projects that you want in a new project.  But for this project, I wanted to practice starting completely from scratch.  There are a couple of good options.  

  

<h4>

</p>

Gradle init

</h4>

</p>

<div>

</p>

I like gradle as a build tool, and gradle has a built in [project initializer](https://docs.gradle.org/current/userguide/build_init_plugin.html).  <span style="text-align: center;">It supports a few project archtypes, including pom (converting a maven project to gradle), java library, and java application.  It even explicitly supports spock testing framework.</span>  

  

</div>

</p>

<div>

</p>

``` brush:
$ gradle init --type java-application --test-framework spockBUILD SUCCESSFUL in 0s2 actionable tasks: 2 executed$ tree.|____build.gradle|____gradle| |____wrapper| | |____gradle-wrapper.jar| | |____gradle-wrapper.properties|____gradlew|____gradlew.bat|____settings.gradle|____src| |____main| | |____java| | | |____App.java| |____test| | |____groovy| | | |____AppTest.groovy
```

</p>

  

</div>

</p>

<div>

</p>

You can see it also generates a demo App and AppTest.

</div>

</p>

  

<h4>

</p>

Spring Boot Initializr

</h4>

</p>

<div>

</p>

For Spring Boot apps, Spring provides the [Spring Boot Initializr](https://start.spring.io/).  This lets you choose from a curated  (but extensive) set of options and dependencies, and then generates a project in a zip file for download.  Similarly to gradle init, it includes a default basic app and test.  

  

<a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj8bg08ZiQIMBzQmfR5-9V08E5PRfr-wtOW4zIVNUgHEg5fDQgvVMmEA58LE4l7QYBEVP-xacLgxFW3O6scYvZEav7tlIwgBURJxDSM6ONj_GgQZgz1LkdbYhUzbyhDxeAum80acqlVi4k/s1600/SpringBootInitializr.png" data-imageanchor="1" style="clear: left; margin-bottom: 1em; margin-right: 1em; text-align: center;"><img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj8bg08ZiQIMBzQmfR5-9V08E5PRfr-wtOW4zIVNUgHEg5fDQgvVMmEA58LE4l7QYBEVP-xacLgxFW3O6scYvZEav7tlIwgBURJxDSM6ONj_GgQZgz1LkdbYhUzbyhDxeAum80acqlVi4k/s640/SpringBootInitializr.png" data-border="0" data-original-height="436" data-original-width="1024" width="640" height="272" /></a>  

  

</div>

</p>

<div>

</p>

``` brush:
$ unzip demo.zipArchive:  demo.zip   creating: demo/  inflating: demo/gradlew   creating: demo/gradle/   creating: demo/gradle/wrapper/   creating: demo/src/   creating: demo/src/main/   creating: demo/src/main/java/   creating: demo/src/main/java/net/   creating: demo/src/main/java/net/ryanmckay/   creating: demo/src/main/java/net/ryanmckay/demo/   creating: demo/src/main/resources/   creating: demo/src/main/resources/static/   creating: demo/src/main/resources/templates/   creating: demo/src/test/   creating: demo/src/test/java/   creating: demo/src/test/java/net/   creating: demo/src/test/java/net/ryanmckay/   creating: demo/src/test/java/net/ryanmckay/demo/  inflating: demo/.gitignore  inflating: demo/build.gradle  inflating: demo/gradle/wrapper/gradle-wrapper.jar  inflating: demo/gradle/wrapper/gradle-wrapper.properties  inflating: demo/gradlew.bat  inflating: demo/src/main/java/net/ryanmckay/demo/DemoApplication.java  inflating: demo/src/main/resources/application.properties  inflating: demo/src/test/java/net/ryanmckay/demo/DemoApplicationTests.java
```

</p>

  

</div>

</p>

<div>

</p>

Pretty much the only thing I don't like here is that the test isn't spock and there doesn't seem to be a way to choose it. Not a big deal, its easy to change afterward.  I went with initializr for this project.

</div>

</p>

  

<h4>

</p>

JHipster

</h4>

</p>

<div>

</p>

[JHipster](https://jhipster.github.io/) is an opinionated full-stack project generator for Spring Boot + Angular apps.  It has a lot of features that I want to explore later, so for now I stuck with Spring Boot Initializr.  

  

<h2>

</p>

Add a Rest Controller

</h2>

</p>

I could have gone straight to CI at this point, but I wanted to do a little extra work that I wish the Initializr could have done for me.  I copied in a basic hello world spring rest controller (and its tests) from the [Spring Restful Web Service Guide](http://spring.io/guides/gs/rest-service/).  The repo is now at this point <https://github.com/ryanmckaytx/java-docker-example/tree/v0.1>

</div>

</p>
