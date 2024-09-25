Title: Docker Java Example Part 2: Spring Web MVC Testing
Date: 2017-08-23 12:47
Author: Ryan McKay
Tags: testing, spring
Slug: docker-java-example-part-2-spring-web
Status: published
Summary: The next step was to add some tests. The tests that came with the demo controller used a Spring feature I was not familiar with, MockMvc. The Spring Guide "[Testing the Web Layer](https://spring.io/guides/gs/testing-web/)" provides a good discussion of various levels of testing, focusing on how much of the Spring context to load.

<div class="toc" markdown="1">
<div class="toctitle">Docker Java Example Series</div>

1. [Initializing a new Spring Boot Project](/docker-java-example-part-1-initializing.html)
2. [Spring Web MVC Testing](/docker-java-example-part-2-spring-web.html)
3. [Transmode Gradle Plugin](/docker-java-example-part3-transmode-gradle-plugin.html)
4. [Bmuschko and Nebula Gradle Plugins](/docker-java-example-part-4-bmuschko-nebula-gradle-docker-plugins.html)
5. [Kubernetes](/docker-java-example-part-5-kubernetes.html)
</div>

The next step was to add some tests. The tests that came with the demo controller used a Spring feature I was not familiar with, MockMvc. The Spring Guide "[Testing the Web Layer](https://spring.io/guides/gs/testing-web/)" provides a good discussion of various levels of testing, focusing on how much of the Spring context to load. There are 3 main levels: 

1. start the full Tomcat server with full Spring context,
2. full Spring context without server, and 
3. narrower MVC-focused context without server. 

I wanted to compare all three, plus add in variation in testing framework and assertion framework. 

Specifically I wanted to add [Spock](http://spockframework.org/spock/docs/1.0/spock_primer.html) with groovy power assert.  The aspects I wanted to compare were: test speed, readability of test code, readability of test output.  I intentionally made one of the tests fail in each approach to compare output.  

![Spring logo]({static}/images/spring-300x293.png "Spring")

## Spock with Full Tomcat Server
This is the approach I am most familiar with.  

[GreetingControllerSpec.groovy](https://github.com/ryanmckaytx/java-docker-example/blob/v0.2/src/test/groovy/net/ryanmckay/demo/GreetingControllerSpec.groovy)

### Timing
I ran and timed the test in isolation with

``` bash
$ ./gradlew test --tests '*GreetingControllerSpec' --profile
```

Total 'test' task time (reported by gradle profile output): 13.734s  
Total test run time (reported by junit test output): 12.690s  
Time to start GreetingControllerSpec (load full context and start tomcat): 12.157s  
So, not fast. Maybe one of the other approaches can do better.

### Test Code Readability

``` groovy
def "no Param greeting should return default message"() {

    when:
    ResponseEntity<Greeting> responseGreeting = restTemplate
                .getForEntity("http://localhost:" + port + "/greeting", Greeting.class)

    then:
    responseGreeting.statusCode == HttpStatus.OK
    responseGreeting.body.content == "blah"
}
```
I really like Spock. I like the plain English test names. I like the separate sections for given, when, then, etc. I think it reads well and makes it obvious what is under test.

### Test Output Readability
When a test fails, you want to see why, right?  In this aspect, [groovy power assertions](http://groovy-lang.org/testing.html#_power_assertions) are simply unparalleled.

``` 
Condition not satisfied:

responseGreeting.body.content == "blah"
|                |    |       |
|                |    |       false
|                |    |       12 differences (7% similarity)
|                |    |       (He)l(lo, World!)
|                |    |       (b-)l(ah--------)
|                |    Hello, World!
|                Greeting(id=1, content=Hello, World!)
<200 OK,Greeting(id=1, content=Hello, World!),{Content-Type=[application/json;charset=UTF-8], Transfer-Encoding=[chunked], Date=[Tue, 22 Aug 2017 22:06:33 GMT]}>

 at net.ryanmckay.demo.GreetingControllerSpec.no Param greeting should return default message(GreetingControllerSpec.groovy:27)
```

Note that the nice output for responseGreeting itself comes from ResponseEntity.toString(), and from Greeting.toString(), which is provided by [Lombok](https://projectlombok.org/).  

## Spock with MockMvc
By adding @AutoConfigureMockMvc to your test class, you can inject a MockMvc instance, which facilitates making calls directly to Springs HTTP request handling layer.  This allows you to skip starting up a Tomcat server, so should save some time and/or memory.  On the other hand, you are testing less of the round trip, so the time savings would need to be significant to justify this approach.  

[GreetingControllerMockMvcSpec.groovy](https://github.com/ryanmckaytx/java-docker-example/blob/v0.2/src/test/groovy/net/ryanmckay/demo/GreetingControllerMockMvcSpec.groovy)

### Timing
This approach was about 500ms faster than with tomcat.  Not significant enough to justify for me, considering the overall time scale.  

Total 'test' task time (reported by gradle profile output): 13.263s  
Total test run time (reported by junit test output): 12.281s   
Time to start GreetingControllerSpec (load full context, no tomcat): 11.804s

### Test Code Readability
```
def "no Param greeting should return default message"() {

    when:
    def resultActions = mockMvc.perform(get("/greeting")).andDo(print())

    then:
    resultActions
            .andExpect(status().isOk())
    
```

This reads reasonably well.  Capturing the resultActions in the *when* block to use later in the *then* block is a little awkward, but not too bad.  Being able to express arbitrary JSON path expectations is convenient.  I didn't see an obvious way to get a ResponseEntity as was done in the full Tomcat example.

### Test Output Readability
``` 
Condition failed with Exception:

resultActions .andExpect(status().isOk()) .andExpect(jsonPath('$.content').value("blah"))
|              |         |        |        |         |                     |
|              |         |        |        |         |                     org.springframework.test.web.servlet.result.JsonPathResultMatchers$2@1f977413
|              |         |        |        |         org.springframework.test.web.servlet.result.JsonPathResultMatchers@6cd50e89
|              |         |        |        java.lang.AssertionError: JSON path "$.content" expected:<blah> but was:<Hello, World!&rt;
|              |         |        org.springframework.test.web.servlet.result.StatusResultMatchers$10@660dd332
|              |         org.springframework.test.web.servlet.result.StatusResultMatchers@251379e8
|              org.springframework.test.web.servlet.MockMvc$1@68837646
org.springframework.test.web.servlet.MockMvc$1@68837646

 at net.ryanmckay.demo.GreetingControllerMockMvcSpec.no Param greeting should return default message(GreetingControllerMockMvcSpec.groovy:29)
Caused by: java.lang.AssertionError: JSON path "$.content" expected:<blah> but was:<Hello, World!&rt;
```

This test output does not read well at all. Spock and the Spring MockMvc library are both tripping over each other trying to provide verbose output.  I think you need choose either Spock or MockMvc, but not both.  

## JUnit with WebMvcTest and MockMvc
This configuration is on the far other end of the spectrum from full service Spock.  With @WebMvcTest, not only does it not start a Tomcat server, it doesn't even load a full context.  In the current state of the project this doesn't make much of a difference because the GreetingController has no injected dependencies.  If it did, I would have to mock those out.  Again, because of the differences from "real" configuration, time savings would need to be significant.  

[GreetingControllerTests.java](https://github.com/ryanmckaytx/java-docker-example/blob/v0.2/src/test/groovy/net/ryanmckay/demo/GreetingControllerTests.java)

### Timing
This approach was also about 500ms faster overall than full context with Tomcat.  

Total 'test' task time (reported by gradle profile output): 13.275s  
Total test run time (reported by junit test output): 0.269s   
Time to start GreetingControllerSpec (load narrow context, no tomcat): 11.88s

### Test Code Readability
``` java
@Test
public void noParamGreetingShouldReturnDefaultMessage() throws Exception {

    this.mockMvc.perform(get("/greeting")).andDo(print()).andExpect(status().isOk())
            .andExpect(jsonPath("$.content").value("blah"));
}
```

This is the least readable for me.  Again, I like separating the call under test from the assertions.  

### Test Output Readability
The failure message for MockMvc-based assertion failures isn't as informative as Spock in this case.

``` 
java.lang.AssertionError: JSON path "$.content" expected:<blah> but was:<Hello, World!>" type="java.lang.AssertionError">java.lang.AssertionError: JSON path "$.content" expected:<blah> but was:<Hello, World!>
```

Because the test called .andDo(print()), some additional information is available in the standard out of the test, including the full response status code and body.  

## Conclusion
I'm as convinced as ever that Spock is the premier Java testing framework.  I'm reserving judgment on the Spring annotations that let you avoid starting a Tomcat server or load the full context.  If the project gets more complicated, those could potentially provide a nice speedup.  

I tagged the [code repo at v0.2](https://github.com/ryanmckaytx/java-docker-example/tree/v0.2) at this point.