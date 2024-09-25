Title: Docker Java Example Part 1: Initializing a new Spring Boot Project
Date: 2017-07-23 21:12
Author: Ryan McKay
Tags: gradle, spring
Slug: docker-java-example-part-1-initializing
Status: published
Summary: I've been wanting to learn more about Docker for a while.  I'm almost done with this udemy course [Docker for Java Developers](https://www.udemy.com/docker-for-java-developers/learn/v4/overview).  It's a good course, and the concepts are straightforward.  To help me commit it to memory, I wanted to do my own project to apply what I'm learning.  

<div class="toc" markdown="1">
<div class="toctitle">Docker Java Example Series</div>

1. [Initializing a new Spring Boot Project](/docker-java-example-part-1-initializing.html)
2. [Spring Web MVC Testing](/docker-java-example-part-2-spring-web.html)
3. [Transmode Gradle Plugin](/docker-java-example-part3-transmode-gradle-plugin.html)
4. [Bmuschko and Nebula Gradle Plugins](/docker-java-example-part-4-bmuschko-nebula-gradle-docker-plugins.html)
5. [Kubernetes](/docker-java-example-part-5-kubernetes.html)
</div>

I've been wanting to learn more about Docker for a while.  I'm almost done with this udemy course [Docker for Java Developers](https://www.udemy.com/docker-for-java-developers/learn/v4/overview).  Its a good course, and the concepts are straightforward.  To help me commit it to memory, I wanted to do my own project to apply what I'm learning.  

<https://github.com/ryanmckaytx/java-docker-example>  

In this part, I'm just going to initialize a new project.  [Part 2](http://againstentropy.blogspot.com/2017/08/docker-java-example-part-2-spring-web.html) covers Spring Web MVC testing.  

## Set up your dev machine
Every once in a while, like when you switch jobs and get a new laptop, you need to set up a machine for java development.  

![SDKMan logo]({static}/images/sdk-man-logo.png "SDKMan")

For this, I like to use [sdkman](http://sdkman.io/).  It helps install the tools you need to do java development.  It also helps switch between multiple versions of those tools.  I've installed java, groovy, gradle, and maven.

``` bash
$ sdk current

Using:

gradle: 4.0
groovy: 2.4.11
java: 8u131-zulu
maven: 3.5.0
```

## Create a new project
There are a few good ways to do this.  The typical way I do this is to copy another project.  Within an organization, or at least within a team, there is typically some amount of infrastructure and institutional knowledge built into existing projects that you want in a new project.  But for this project, I wanted to practice starting completely from scratch.  There are a couple of good options.  

### Gradle init
I like gradle as a build tool, and gradle has a built in [project initializer](https://docs.gradle.org/current/userguide/build_init_plugin.html). 
It supports a few project archtypes, including pom (converting a maven project to gradle), java library, and java application.  It even explicitly supports spock testing framework.

``` bash
$ gradle init --type java-application --test-framework spock

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
$ tree .
|____build.gradle
|____gradle
| |____wrapper
| | |____gradle-wrapper.jar
| | |____gradle-wrapper.properties
|____gradlew
|____gradlew.bat
|____settings.gradle
|____src
| |____main
| | |____java
| | | |____App.java
| |____test
| | |____groovy
| | | |____AppTest.groovy
```

You can see it also generates a demo App and AppTest.

### Spring Boot Initializr
For Spring Boot apps, Spring provides the [Spring Boot Initializr](https://start.spring.io/).  This lets you choose from a curated  (but extensive) set of options and dependencies, and then generates a project in a zip file for download.  Similarly to gradle init, it includes a default basic app and test.  

![Spring Boot Initializr logo]({static}/images/SpringBootInitializr.png "Spring Boot Initializr")
  
``` bash
$ unzip demo.zip
Archive:  demo.zip
   creating: demo/
  inflating: demo/gradlew
   creating: demo/gradle/
   creating: demo/gradle/wrapper/
   creating: demo/src/
   creating: demo/src/main/
   creating: demo/src/main/java/
   creating: demo/src/main/java/net/
   creating: demo/src/main/java/net/ryanmckay/
   creating: demo/src/main/java/net/ryanmckay/demo/
   creating: demo/src/main/resources/
   creating: demo/src/main/resources/static/
   creating: demo/src/main/resources/templates/
   creating: demo/src/test/
   creating: demo/src/test/java/
   creating: demo/src/test/java/net/
   creating: demo/src/test/java/net/ryanmckay/
   creating: demo/src/test/java/net/ryanmckay/demo/
  inflating: demo/.gitignore
  inflating: demo/build.gradle
  inflating: demo/gradle/wrapper/gradle-wrapper.jar
  inflating: demo/gradle/wrapper/gradle-wrapper.properties
  inflating: demo/gradlew.bat
  inflating: demo/src/main/java/net/ryanmckay/demo/DemoApplication.java
  inflating: demo/src/main/resources/application.properties
  inflating: demo/src/test/java/net/ryanmckay/demo/DemoApplicationTests.java
```

Pretty much the only thing I don't like here is that the test isn't spock and there doesn't seem to be a way to choose it. Not a big deal, its easy to change afterward.  I went with initializr for this project.

### JHipster
[JHipster](https://jhipster.github.io/) is an opinionated full-stack project generator for Spring Boot + Angular apps.  It has a lot of features that I want to explore later, so for now I stuck with Spring Boot Initializr.  

## Add a Rest Controller
I could have gone straight to CI at this point, but I wanted to do a little extra work that I wish the Initializr could have done for me.  I copied in a basic hello world spring rest controller (and its tests) from the [Spring Restful Web Service Guide](http://spring.io/guides/gs/rest-service/).  The repo is now at this point <https://github.com/ryanmckaytx/java-docker-example/tree/v0.1>