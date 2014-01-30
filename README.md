# Ursus

### Overview

Ursus is a [Dropwizard](http://dropwizard.codahale.com/) inspired framework for developing lightweight
web services on the JVM and [Grizzly](https://grizzly.java.net/). Ursus combines Grizzly's
NIO/Event Driven framework and popular libraries such as [Jersey 2.5](https://jersey.java.net/) and
[Project Tyrus - JSR 356: Java API for WebSocket - Reference Implementation](https://tyrus.java.net/) with
the simple conventions found in Dropwizard, allowing you to focus on developing your services rather
than managing their accompanying dependencies, reducing time-to-market and maintenance.

#### Grizzly HTTP Container

Grizzly is a multi-protocol framework built on top of Java NIO. Grizzly aims to simplify development of
robust and scalable servers. Jersey provides a container extension module that enables support for using
Grizzly as a plain vanilla HTTP container that runs JAX-RS applications. Starting a Grizzly server to
run a JAX-RS or Jersey application is one of the most lightweight and easy ways how to expose a
functional RESTful services application.

#### REST / Jersey

Jersey is an open source, production quality, framework for developing RESTful Web Services in Java that
provides support for JAX-RS APIs and serves as a JAX-RS (JSR 311 & JSR 339) Reference Implementation.
Developing RESTful Web services with Jersey is a snap and the latest Jersey implementation included in
Ursus provides support for both asynchronous clients and server side services with [AsyncResponse](https://jax-rs-spec.java.net/nonav/2.0/apidocs/javax/ws/rs/container/AsyncResponse.html)

#### WebSockets / Tyrus

Tyrus is the open source [JSR 356](https://java.net/projects/websocket-spec) Java reference implementation of
WebSockets. The WebSocket protocol provides full-duplex communications between server and remote hosts.
WebSocket is designed to be implemented in web browsers and web servers, but it can be used by any client or server application.
The WebSocket protocol makes possible more interaction between a browser and a web site, facilitating live content
and the creation of real-time applications and services.

#### Additional Libraries

Ursus also includes many libraries found in Dropwizard and other frameworks to help speed up development including,

* [Guava](https://code.google.com/p/guava-libraries/)
* [Joda time](www.joda.org/joda-time/)
* [Hibernate Validator](http://hibernate.org/validator/)
* [Jackson](http://jackson.codehaus.org/) for YAML and JSON support]
* [Logback](http://logback.qos.ch/) and [SLF4J](http://www.slf4j.org/) for bridging of Grizzly's java.util.logging
* An [Apache HTTPClient](http://hc.apache.org/httpclient-3.x/) and a [Jersey 2.5 client](https://jersey.java.net/documentation/latest/user-guide.html#client)

#### Using Ursus
Ursus releases artifacts are hosted on the central repository, to get started with ursus simply add the ```ursus-core```
module to your pom.xml as a dependency.

```xml
 <dependencies>
    <dependency>
        <groupId>com.aceevo</groupId>
        <artifactId>ursus-core</artifactId>
        <version>0.2</version>
    </dependency>
 </dependencies>
```

### Jersey Application Quick Start
Let's take a look at the ```ursus-example-application``` project a simple HelloWorld service. It's fully implemented, but we're going
to walk through the steps it took to build this project. If it helps you might want to clone the Ursus repro, [https://github.com/rjenkins/ursus.git](https://github.com/rjenkins/ursus.git)
and modify ```ursus-example-application``` as we progress through the quick start building up the application yourself.

Ursus applications require 2 classes, a subclass of ```UrsusJerseyApplication``` and a subclass of ```UrsusJerseyApplicationConfiguration```. The com.aceevo.ursus.example package contains both.
Let's start by looking at the ```ExampleApplicationConfiguration``` class.

```java
package com.aceevo.ursus.example;

import com.aceevo.ursus.config.UrsusJerseyApplicationConfiguration;
import com.fasterxml.jackson.annotation.JsonProperty;
import org.hibernate.validator.constraints.NotEmpty;

public class ExampleApplicationConfiguration extends UrsusJerseyApplicationConfiguration {

    @NotEmpty
    @JsonProperty
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
Just like Dropwizard we will specify our environment specific configuration parameters in a [YAML](http://www.yaml.org/) configuration
file and these parameters will be deserialized by Jackson and validated. By default Ursus will look for a file whose name is the class name of
your application with a ```.yml``` extension, for this example application that file is named ```exampleapplication.yml```. If you wish to specify
a different location for you configuration you can specify that on that with arguments.

```java -jar <path to jar file> server <path to your YAML file```

In addition to your own environment specific configuration parameters [UrsusJerseyApplicationConfiguration](https://github.com/rjenkins/ursus/blob/master/ursus-config/src/main/java/com/aceevo/ursus/config/UrsusJerseyApplicationConfiguration.java)
defines a large set of configuration properties that allow you to modify all of the granular configuration options available with Grizzly and many of the other
included libraries simply by adding lines of YAML (more on that later).

#### Creating our YAML File

Let's allow our ExampleApplication the ability to specify who it should say Hello to. For this we'll use the name property we created in
```ExampleApplicationConfiguration```. Now we can define this parameter in ```exampleapplication.yml```

```yaml
httpServer:
  host: localhost
name: Ray
```

Great! Let's move on to creating our first application.

#### Creating an UrsusJerseyApplication

We'll name our application ```ExampleApplication``` and subclass ```UrsusJerseyApplication``` we also need to parameterize
ExampleApplication with the type of our UrsusJerseyApplicationConfiguration class, that's named ```ExampleApplicationConfiguration```.
If your using an IDE like IntelliJ you can select implement methods and you'll get the skeleton of our application. We also
need to add a ```public static void main(String[] args)``` method and instantiate an instance of ExampleApplication.

```java
package com.aceevo.ursus.example;

import com.aceevo.ursus.core.UrsusJerseyApplication;
import org.glassfish.grizzly.http.server.HttpServer;

public class ExampleApplication extends UrsusJerseyApplication<ExampleApplicationConfiguration> {

    protected ExampleApplication(String[] args) {
        super(args);
    }

    public static void main(String[] args) throws Exception {
        new ExampleApplication(args);
    }

    @Override
    protected void boostrap(ExampleApplicationConfiguration exampleApplicationConfiguration) {
        //To change body of implemented methods use File | Settings | File Templates.
    }

    @Override
    protected void run(HttpServer httpServer) {
        //To change body of implemented methods use File | Settings | File Templates.
    }
}
```

ExampleApplication in the ```ursus-example-application``` project is filled out and have more data than this, but let's ignore that for now.
If you like you can always overwrite ```ExampleApplication``` with the above and follow


#### Building our first artifact and running our application.

Like Dropwizard Ursus recommends building a "fat" jar that contains all the ```.class``` files needed to run our service.
For the ```ursus-example-application``` we've already added this to the pom.xml, here's the relavent parts of our configuration.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>2.1</version>
            <configuration>
                <createDependencyReducedPom>true</createDependencyReducedPom>
                <filters>
                    <filter>
                        <artifact>*:*</artifact>
                        <excludes>
                            <exclude>META-INF/*.SF</exclude>
                            <exclude>META-INF/*.DSA</exclude>
                            <exclude>META-INF/*.RSA</exclude>
                        </excludes>
                    </filter>
                </filters>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>com.aceevo.ursus.example.ExampleApplication</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
            ...
```

If you've cloned the ursus git repo and have modified the ```ursus-example-application``` we should be able to build and test
our project, build by running ```mvn clean; mvn package``` from the ursus director. Now we can cd into ursus-example-application and
start our server with the following command ```java -jar ./target/ursus-example-application-0.2-SNAPSHOT.jar```

Unfortunately our application starts and then quickly exists, what's up?

```
boundray:ursus-example-application rayjenkins$ java -jar ./target/ursus-example-application-0.2-SNAPSHOT.jar
19:16:17.488 [main] INFO  o.h.validator.internal.util.Version - HV000001: Hibernate Validator 5.0.1.Final
19:16:18.009 [main] INFO  o.g.jersey.server.ApplicationHandler - Initiating Jersey application, version Jersey: 2.5 2013-12-18 14:27:29...
boundray:ursus-example-application rayjenkins$
```

#### Starting the HttpServer

We need to start our HttpServer instance to start our application. ```protected void run(HttpServer httpServer)``` method passes us
a configured HttpServer (after bootstrap has been called) allowing us to make any additional modifications to the Grizzly Http Server before
we start our application. When we're ready to start our application Ursus provides a helper method ```protected void startWithShutdownHook(final HttpServer httpServer)```
that handles starting the HttpServer and registering a shutdown hook for our application.

```java
/**
 * Convenience method for starting this Grizzly HttpServer
 *
 * @param httpServer
 */
protected void startWithShutdownHook(final HttpServer httpServer) {

    Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
        @Override
        public void run() {
            LOGGER.info("Stopping Grizzly HttpServer...");
            httpServer.stop();
            LOGGER.info("Stopping all managed services...");
            for (Service service : managedServices) {
                service.stopAsync();
            }
        }
    }, "shutdownHook"));

    try {
        LOGGER.info("Starting all managed services...");
        for (Service service : managedServices) {
            service.startAsync();
        }
        httpServer.start();
        printBanner(getClass().getSimpleName());
        LOGGER.info("Press CTRL^C to exit..");
        Thread.currentThread().join();
    } catch (Exception e) {
        LOGGER.error("There was an error while starting Grizzly HTTP server.", e);
    }
}
```

Let's add that to our ExampleApplication run method, build again and retry.

```java
...

@Override
protected void run(HttpServer httpServer) {
    startWithShutdownHook(httpServer);
}

...
```

```java -jar ./target/ursus-example-application-0.2-SNAPSHOT.jar```

```
boundray:ursus-example-application rayjenkins$ java -jar ./target/ursus-example-application-0.2-SNAPSHOT.jar
19:32:04.643 [main] INFO  o.h.validator.internal.util.Version - HV000001: Hibernate Validator 5.0.1.Final
19:32:05.150 [main] INFO  o.g.jersey.server.ApplicationHandler - Initiating Jersey application, version Jersey: 2.5 2013-12-18 14:27:29...
19:32:05.504 [main] INFO  c.aceevo.ursus.core.UrsusJerseyApplication - Starting all managed services...
19:32:05.557 [main] INFO  o.g.g.http.server.NetworkListener - Started listener bound to [localhost:8080]
19:32:05.561 [main] INFO  o.g.grizzly.http.server.HttpServer - [HttpServer] Started.
19:32:05.570 [main] INFO  c.aceevo.ursus.core.UrsusJerseyApplication - Starting ExampleApplication
 __  __   ______    ______   __  __   ______
/_/\/_/\ /_____/\  /_____/\ /_/\/_/\ /_____/\
\:\ \:\ \\:::_ \ \ \::::_\/_\:\ \:\ \\::::_\/_
 \:\ \:\ \\:(_) ) )_\:\/___/\\:\ \:\ \\:\/___/\
  \:\ \:\ \\: __ `\ \\_::._\:\\:\ \:\ \\_::._\:\
   \:\_\:\ \\ \ `\ \ \ /____\:\\:\_\:\ \ /____\:\
    \_____\/ \_\/ \_\/ \_____\/ \_____\/ \_____\/

19:32:05.570 [main] INFO  c.aceevo.ursus.core.UrsusJerseyApplication - Press CTRL^C to exit..
```

Success! Now it's time to move on to implementing our service.

#### Representations and Resources
We've got our application started but we need to provide some resources and some representations. As previously mentioned
this Hello World application is going to support specifiying exactly who we're going to say hello to. Let's create a model
class to represent a Hello.

```java
package com.aceevo.ursus.example.model;

import org.codehaus.jackson.annotate.JsonProperty;

public class Hello {

    @JsonProperty
    private String name;

    public Hello() {

    }

    public Hello(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    void setName(String name) {
        this.name = name;
    }
}
```

Now that we've created a representation we can create a Jersey Resource class for representing this resource.

#### Creating Resources

Let's create a simple Jersey resource that represents our Hello representation. In this Resource we'll add support for the
[GET](http://tools.ietf.org/search/rfc2616#section-9.3) and [POST](http://tools.ietf.org/search/rfc2616#section-9.5) methods as well as support for
an [AsyncResponse](https://jersey.java.net/apidocs/latest/jersey/javax/ws/rs/container/AsyncResponse.html) GET.

```java
package com.aceevo.ursus.example.api;

import com.aceevo.ursus.example.ExampleApplicationConfiguration;
import com.aceevo.ursus.example.model.Hello;

import javax.inject.Inject;
import javax.validation.Valid;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.container.AsyncResponse;
import javax.ws.rs.container.Suspended;
import javax.ws.rs.core.*;
import java.net.URI;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

@Path("hello")
@Produces(MediaType.APPLICATION_JSON)
public class HelloWorldResource {

    private final ExecutorService executorService = Executors.newSingleThreadExecutor();

    @Inject
    ExampleApplicationConfiguration exampleApplicationConfiguration;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response hello() {
        return Response.ok(new Hello(exampleApplicationConfiguration.getName())).build();
    }

    @POST
    @Produces(MediaType.APPLICATION_JSON)
    public Response createHello(@Valid Hello hello, @Context UriInfo uriInfo) {
        URI uri = UriBuilder.fromUri(uriInfo.getRequestUri()).path(hello.getName()).build();
        return Response.created(uri).build();
    }

    @GET
    @Path("/async")
    @Produces(MediaType.APPLICATION_JSON)
    public void sayHelloAsync(final @Suspended AsyncResponse asyncResponse) throws Exception {

        executorService.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.currentThread().sleep(5000);
                    asyncResponse.resume(Response.ok(new Hello(exampleApplicationConfiguration.getName())).build());
                } catch (InterruptedException e) {
                    e.printStackTrace();  //To change body of catch statement use File | Settings | File Templates.
                }
            }
        });

    }
}
```

There's a few things going on in this resource, so let's walk through them step by step.

* We define the path and media type at the class level with ```@Path("hello") and @Produces(MediaType.APPLICATION_JSON)```
* We create an ExecutorService for our AsyncResponse resource with ```java private final ExecutorService executorService = Executors.newSingleThreadExecutor();```
* We @Inject an instance of  ```ExampleApplicationConfiguration```

The first ```@GET``` and ```@POST``` methods are pretty standard, let's take a look at the ```@PATH("/async") sayHelloAsync``` resource method. This method
uses the ```@Suspend``` annotation and allows us to suspend the current request releasing the thread responsible for handling the request and resume
asynchronously once we're ready to response.

#### Registering Resources

Now that we've created our first resource we need to register then with our Jersey ResourceConfig, there are many ways to do this (more on this later)
but for our example application let's use Jersey's built in resource discovery and specify the package where our resources exist. We'll do this in the bootstrap method of ```ExampleApplication```.

```java
package com.aceevo.ursus.example;

import com.aceevo.ursus.core.UrsusJerseyApplication;
import com.aceevo.ursus.example.api.EchoEndpointResource;
import com.google.common.collect.ImmutableMap;
import com.google.common.collect.Maps;
import org.glassfish.grizzly.http.server.HttpServer;

import javax.websocket.server.ServerEndpointConfig;

public class ExampleApplication extends UrsusJerseyApplication<ExampleApplicationConfiguration> {

    ...

    @Override
    protected void boostrap(ExampleApplicationConfiguration exampleApplicationConfiguration) {
        packages("com.aceevo.ursus.example.api");
    }

    @Override
    protected void run(HttpServer httpServer) {
        startWithShutdownHook(httpServer);
    }
}
```
Now we're ready to build and try out our resources.

```
boundray:ursus-example-application rayjenkins$ java -jar ./target/ursus-example-application-0.2-SNAPSHOT.jar
21:27:58.901 [main] INFO  o.h.validator.internal.util.Version - HV000001: Hibernate Validator 5.0.1.Final
21:27:59.901 [main] INFO  o.g.jersey.server.ApplicationHandler - Initiating Jersey application, version Jersey: 2.5 2013-12-18 14:27:29...
21:28:00.290 [main] INFO  c.aceevo.ursus.core.UrsusJerseyApplication - Starting all managed services...
21:28:00.342 [main] INFO  o.g.g.http.server.NetworkListener - Started listener bound to [localhost:8080]
21:28:00.357 [main] INFO  o.g.grizzly.http.server.HttpServer - [HttpServer] Started.
21:28:00.366 [main] INFO  c.aceevo.ursus.core.UrsusJerseyApplication - Starting ExampleApplication
 __  __   ______    ______   __  __   ______
/_/\/_/\ /_____/\  /_____/\ /_/\/_/\ /_____/\
\:\ \:\ \\:::_ \ \ \::::_\/_\:\ \:\ \\::::_\/_
 \:\ \:\ \\:(_) ) )_\:\/___/\\:\ \:\ \\:\/___/\
  \:\ \:\ \\: __ `\ \\_::._\:\\:\ \:\ \\_::._\:\
   \:\_\:\ \\ \ `\ \ \ /____\:\\:\_\:\ \ /____\:\
    \_____\/ \_\/ \_\/ \_____\/ \_____\/ \_____\/

21:28:00.366 [main] INFO  c.aceevo.ursus.core.UrsusJerseyApplication - Press CTRL^C to exit..
^Z
[1]+  Stopped                 java -jar ./target/ursus-example-application-0.2-SNAPSHOT.jar
boundray:ursus-example-application rayjenkins$ bg
[1]+ java -jar ./target/ursus-example-application-0.2-SNAPSHOT.jar &
```
```
boundray:ursus-example-application rayjenkins$ curl -v -X GET -H "Accept: application/json" http://localhost:8080/hello
* About to connect() to localhost port 8080 (#0)
*   Trying ::1...
* Connection refused
*   Trying 127.0.0.1...
* connected
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /hello HTTP/1.1
> User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8y zlib/1.2.5
> Host: localhost:8080
> Accept: application/json
>
< HTTP/1.1 200 OK
< Content-Type: application/json
< Date: Thu, 16 Jan 2014 05:28:07 GMT
< Content-Length: 14
<
* Connection #0 to host localhost left intact
{"name":"Ray"}* Closing connection #0
```

```
boundray:ursus-example-application rayjenkins$ curl -v -H "Content-Type: application/json" -X POST http://localhost:8080/hello -d '{ "name" : "Andrea" }'
* About to connect() to localhost port 8080 (#0)
*   Trying ::1...
* Connection refused
*   Trying 127.0.0.1...
* connected
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST /hello HTTP/1.1
> User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8y zlib/1.2.5
> Host: localhost:8080
> Accept: */*
> Content-Type: application/json
> Content-Length: 21
>
* upload completely sent off: 21 out of 21 bytes
< HTTP/1.1 201 Created
< Location: http://localhost:8080/hello/Andrea
< Date: Thu, 16 Jan 2014 05:28:11 GMT
< Content-Length: 0
<
* Connection #0 to host localhost left intact
* Closing connection #0
```

```
boundray:ursus-example-application rayjenkins$ curl -v -X GET -H "Accept: application/json" http://localhost:8080/hello/async
* About to connect() to localhost port 8080 (#0)
*   Trying ::1...
* Connection refused
*   Trying 127.0.0.1...
* connected
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /hello/async HTTP/1.1
> User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8y zlib/1.2.5
> Host: localhost:8080
> Accept: application/json
>
< HTTP/1.1 200 OK
< Content-Type: application/json
< Date: Thu, 16 Jan 2014 05:28:53 GMT
< Content-Length: 14
<
* Connection #0 to host localhost left intact
{"name":"Ray"}* Closing connection #0
```

Ursus supports full content negotiation via Jersey, so if your resource class produces ```application/json``` but the client accepts ```text/plain```,
Jersey will reply with a ```406 Not Acceptable```.

```
boundray:ursus-example-application rayjenkins$ curl -v -H "Accept: text/plain" -X GET http://localhost:8080/hello
* About to connect() to localhost port 8080 (#0)
*   Trying ::1...
* Connection refused
*   Trying 127.0.0.1...
* connected
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /hello HTTP/1.1
> User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8y zlib/1.2.5
> Host: localhost:8080
> Accept: vext/plain
>
< HTTP/1.1 406 Not Acceptable
< Content-Type: text/plain
< Date: Thu, 16 Jan 2014 07:56:36 GMT
< Content-Length: 2122
<
javax.ws.rs.NotAcceptableException: HTTP 406 Not Acceptable
	at org.glassfish.jersey.server.internal.routing.MethodSelectingRouter.getMethodRouter(MethodSelectingRouter.java:573)
	at org.glassfish.jersey.server.internal.routing.MethodSelectingRouter.access$400(MethodSelectingRouter.java:97)
	at org.glassfish.jersey.server.internal.routing.MethodSelectingRouter$4.apply(MethodSelectingRouter.java:813)
	at org.glassfish.jersey.server.internal.routing.MethodSelectingRouter.apply(MethodSelectingRouter.java:420)
	at org.glassfish.jersey.server.internal.routing.RoutingStage._apply(RoutingStage.java:128)
	at org.glassfish.jersey.server.internal.routing.RoutingStage._apply(RoutingStage.java:131)
	at org.glassfish.jersey.server.internal.routing.RoutingStage._apply(RoutingStage.java:131)
	at org.glassfish.jersey.server.internal.routing.RoutingStage.apply(RoutingStage.java:110)
	at org.glassfish.jersey.server.internal.routing.RoutingStage.apply(RoutingStage.java:65)
	at org.glassfish.jersey.process.internal.Stages.process(Stages.java:197)
	at org.glassfish.jersey.server.ServerRuntime$1.run(ServerRuntime.java:250)
	at org.glassfish.jersey.internal.Errors$1.call(Errors.java:271)
	at org.glassfish.jersey.internal.Errors$1.call(Errors.java:267)
	at org.glassfish.jersey.internal.Errors.process(Errors.java:315)
	at org.glassfish.jersey.internal.Errors.process(Errors.java:297)
	at org.glassfish.jersey.internal.Errors.process(Errors.java:267)
	at org.glassfish.jersey.process.internal.RequestScope.runInScope(RequestScope.java:318)
	at org.glassfish.jersey.server.ServerRuntime.process(ServerRuntime.java:236)
	at org.glassfish.jersey.server.ApplicationHandler.handle(ApplicationHandler.java:1010)
	at org.glassfish.jersey.grizzly2.httpserver.GrizzlyHttpContainer.service(GrizzlyHttpContainer.java:363)
	at org.glassfish.grizzly.http.server.HttpHandler$1.run(HttpHandler.java:217)
	at org.glassfish.grizzly.threadpool.AbstractThreadPool$Worker.doWork(AbstractThreadPool.java:565)
	at org.glassfish.grizzly.threadpool.AbstractThreadPool$Worker.run(AbstractThreadPool.java:545)
	at java.lang.Thread.run(Thread.java:744)
* Connection #0 to host localhost left intact
* Closing connection #0
```

#### Writing Resource Tests

Jersey comes with a testframework provider for Grizzly, let's add that to our pom.xml so we can write some tests.

```xml
<dependency>
    <groupId>org.glassfish.jersey.test-framework.providers</groupId>
    <artifactId>jersey-test-framework-provider-grizzly2</artifactId>
    <version>2.5</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.test-framework</groupId>
    <artifactId>jersey-test-framework-core</artifactId>
    <version>2.5</version>
    <exclusions>
        <exclusion>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

We can extend JerseyTest and override the configure method to create a new grizzly/jersey test environment. We'll
create a dummy ```ExampleApplicationConfiguration``` class and register an UrsusApplicationBinder to support injection
of our configuration as well as register our HelloWordResource class.

```java
package com.aceevo.ursus.example.api;

import com.aceevo.ursus.core.UrsusApplicationBinder;
import com.aceevo.ursus.example.ExampleApplicationConfiguration;
import com.aceevo.ursus.example.model.Hello;
import org.glassfish.jersey.server.ResourceConfig;
import org.glassfish.jersey.test.JerseyTest;
import org.junit.Test;

import javax.ws.rs.client.Entity;
import javax.ws.rs.core.Application;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

import static junit.framework.Assert.assertEquals;

public class HelloWorldTest extends JerseyTest {

    @Override
    protected Application configure() {
        ResourceConfig resourceConfig = new ResourceConfig();
        ExampleApplicationConfiguration exampleApplicationConfiguration = new ExampleApplicationConfiguration();
        exampleApplicationConfiguration.setName("Ray");
        resourceConfig.registerInstances(new UrsusApplicationBinder(exampleApplicationConfiguration));
        resourceConfig.register(HelloWorldResource.class);
        return resourceConfig;
    }

    @Test
    public void helloTest() {
        Response response =  target("/hello").request().get();
        assertEquals(200, response.getStatus());
        assertEquals("Ray", response.readEntity(Hello.class).getName());
    }

    @Test
    public void sayHello() {
        Entity<Hello> helloEntity = Entity.entity(new Hello("Bob"), MediaType.APPLICATION_JSON_TYPE);
        Response response = target("/hello").request().post(helloEntity);
        assertEquals(201, response.getStatus());
        assertEquals("http://localhost:9998/hello/Bob", response.getLocation().toString());
    }

    @Test
    public void helloAsyncTest() throws Exception {
        Response response = target("/hello").request().async().get().get();
        assertEquals(200, response.getStatus());
        assertEquals("Ray", response.readEntity(Hello.class).getName());
    }
}
```

Now let's run ```mvn test``` and verify our resource class works as intended

```
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.aceevo.ursus.example.api.HelloWorldTest
Jan 19, 2014 8:36:39 PM org.glassfish.jersey.server.ApplicationHandler initialize
INFO: Initiating Jersey application, version Jersey: 2.5 2013-12-18 14:27:29...
Jan 19, 2014 8:36:40 PM org.glassfish.jersey.test.grizzly.GrizzlyTestContainerFactory$GrizzlyTestContainer start
INFO: Starting GrizzlyTestContainer...
Jan 19, 2014 8:36:40 PM org.glassfish.grizzly.http.server.NetworkListener start
INFO: Started listener bound to [localhost:9998]
Jan 19, 2014 8:36:40 PM org.glassfish.grizzly.http.server.HttpServer start
INFO: [HttpServer] Started.
Jan 19, 2014 8:36:40 PM org.glassfish.jersey.test.grizzly.GrizzlyTestContainerFactory$GrizzlyTestContainer stop
INFO: Stopping GrizzlyTestContainer...
Jan 19, 2014 8:36:40 PM org.glassfish.grizzly.http.server.NetworkListener shutdownNow
INFO: Stopped listener bound to [localhost:9998]
Jan 19, 2014 8:36:40 PM org.glassfish.jersey.server.ApplicationHandler initialize
INFO: Initiating Jersey application, version Jersey: 2.5 2013-12-18 14:27:29...
Jan 19, 2014 8:36:40 PM org.glassfish.jersey.test.grizzly.GrizzlyTestContainerFactory$GrizzlyTestContainer start
INFO: Starting GrizzlyTestContainer...
Jan 19, 2014 8:36:40 PM org.glassfish.grizzly.http.server.NetworkListener start
INFO: Started listener bound to [localhost:9998]
Jan 19, 2014 8:36:40 PM org.glassfish.grizzly.http.server.HttpServer start
INFO: [HttpServer-1] Started.
Jan 19, 2014 8:36:40 PM org.glassfish.jersey.test.grizzly.GrizzlyTestContainerFactory$GrizzlyTestContainer stop
INFO: Stopping GrizzlyTestContainer...
Jan 19, 2014 8:36:40 PM org.glassfish.grizzly.http.server.NetworkListener shutdownNow
INFO: Stopped listener bound to [localhost:9998]
Jan 19, 2014 8:36:40 PM org.glassfish.jersey.server.ApplicationHandler initialize
INFO: Initiating Jersey application, version Jersey: 2.5 2013-12-18 14:27:29...
Jan 19, 2014 8:36:40 PM org.glassfish.jersey.test.grizzly.GrizzlyTestContainerFactory$GrizzlyTestContainer start
INFO: Starting GrizzlyTestContainer...
Jan 19, 2014 8:36:40 PM org.glassfish.grizzly.http.server.NetworkListener start
INFO: Started listener bound to [localhost:9998]
Jan 19, 2014 8:36:40 PM org.glassfish.grizzly.http.server.HttpServer start
INFO: [HttpServer-2] Started.
Jan 19, 2014 8:36:40 PM org.glassfish.jersey.test.grizzly.GrizzlyTestContainerFactory$GrizzlyTestContainer stop
INFO: Stopping GrizzlyTestContainer...
Jan 19, 2014 8:36:40 PM org.glassfish.grizzly.http.server.NetworkListener shutdownNow
INFO: Stopped listener bound to [localhost:9998]
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.628 sec

Results :

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 5.103s
[INFO] Finished at: Sun Jan 19 20:36:40 CST 2014
[INFO] Final Memory: 17M/439M
[INFO] ------------------------------------------------------------------------
```

#### WebSocket EndPoints

In addition to Jersey, Ursus includes [Tyrus](https://tyrus.java.net/) is the open source JSR 356 - Java API for WebSocket
reference implementation for easy development of WebSocket applications. You can create Annotated EndPoints and register
them within your Ursus application or create them programatically. If you need to pass dependencies to EndPoints you'll
need to do that programmatically and make use of the EndPoint's [UserProperties](http://docs.oracle.com/javaee/7/api/javax/websocket/EndpointConfig.html#getUserProperties)

Let's create a simple programmatic EndPoint for ```ursus-example-application```. We'll pass in our ```ExampleApplicationConfiguration```
in the onOpen method. Now we can use our ```name``` property to respond to clients.

```java
package com.aceevo.ursus.example.api;

import com.aceevo.ursus.example.ExampleApplicationConfiguration;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.websocket.*;
import java.io.IOException;

public class EchoEndpointResource extends Endpoint {

    private final Logger logger = LoggerFactory.getLogger(EchoEndpointResource.class);
    private ExampleApplicationConfiguration exampleApplicationConfiguration;

    @OnOpen
    public void onOpen(final Session session, EndpointConfig config) {
        exampleApplicationConfiguration = (ExampleApplicationConfiguration) config.getUserProperties().get("exampleApplicationConfiguration");
        session.addMessageHandler(new MessageHandler.Whole<String>() {

            @Override
            public void onMessage(String message) {
                logger.info("Received message from client: " + message);
                try {
                    session.getBasicRemote().sendText("Hello " + exampleApplicationConfiguration.getName());
                } catch (IOException e) {
                    e.printStackTrace();  //To change body of catch statement use File | Settings | File Templates.
                }
            }
        });
    }
}
```

#### Registering EndPoints

Ursus provides some short cuts that allow you to quickly register your endpoints and pass any dependencies to your endpoint.
Let's modify our bootstrap method and use the ```registerEndPoint``` method to add ```EchoEndPointResouce```. We simple need
to pass our endpoint class, the path, and a key for our UserProperties, that key will be "exampleApplicationConfiguration"
and the value is the instance of our ```ExampleApplicationConfiguration``` class.

```java
public class ExampleApplication extends UrsusJerseyApplication<ExampleApplicationConfiguration> {

    ...

    @Override
    protected void boostrap(ExampleApplicationConfiguration exampleApplicationConfiguration) {
        packages("com.aceevo.ursus.example.api");
        registerEndpoint(EchoEndpointResource.class, "/echo", "exampleApplicationConfiguration", exampleApplicationConfiguration);
    }

    @Override
    protected void run(HttpServer httpServer) {
        startWithShutdownHook(httpServer);
    }
}
```