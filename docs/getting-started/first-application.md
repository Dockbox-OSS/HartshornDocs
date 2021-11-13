# Your first application
You will build a service that will accept HTTP GET requests at `http://localhost:8080/greeting`. It will respond with a JSON representation of a greeting, of which you can customize the greeting with an optional `name` parameter in the query string.
```
http://localhost:8080/greeting?name=Guus
```

## Set up your development environment
Before you get started developing your application, you first need to set up the project. You can do so by following the guide ['Setting up your development environment'](./setup.md).

## Create a greeting model
Now that you have set up your environment, you can start creating your service. Begin by thinking about service interactions. The service will handle `GET` requests for `/greeting`, with an optional `name` parameter in the query string. The `GET` request should return a `200 OK` response with a JSON body that represents the greeting. It should resemble the following output:
```json
{
    "content": "Hello, World!"
}
```
To model the greeting representation, create a new Plain Old Java Object (POJO) with fields, constructors, and accessors for the `content` data.
```java title="Greeting.java"
public class Greeting {

    private final String content;

    public Greeting(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }
}
```
!!! tip
    Alternatively to creating the getter and constructor manually, you can use [Project Lombok's](https://projectlombok.org/){:target="_blank"} [@Getter](https://projectlombok.org/features/GetterSetter){:target="_blank"} and [@RequiredArgsConstructor](https://projectlombok.org/features/constructor){:target="_blank"}.

## Create a REST controller
Hartshorn's approach to RESTful web services is based around Hartshorn's service-based architecture. This means HTTP requests are handled by controllers, which are extensions of services. These controllers are identified by the `@RestController` annotation.  
The `GreetingController` seen below handles `GET` requests for `/greeting` by returning a new instance of the `Greeting` class:

??? info "View imports"

    ```java
    import org.dockbox.hartshorn.web.annotations.RequestParam;
    import org.dockbox.hartshorn.web.annotations.RestController;
    import org.dockbox.hartshorn.web.annotations.http.HttpGet;
    ```
```java
@RestController
public class GreetingController {

    @HttpGet("/greeting")
    public Greeting greeting(@RequestParam(value = "name", or = "World") String name) {
        return new Greeting("Hello, %s!".formatted(name)); // (1)
    }
}
```

1.  If no value is provided for `name`, the default value `World` is used instead.

This controller is seemingly simple, but there is plenty of functionality being handled by Hartshorn. Let's break it down step by step.

The `@HttpGet` annotation indicates this method is capable of handling HTTP GET requests to `/greeting`.

!!! info
    `@HttpGet` is an extension of `@HttpRequest`, defaulting the `HttpMethod` to `GET`. There are additional companion annotations for other HTTP methods, like `@HttpPost` for `POST` requests. `@HttpGet` is a synonym for `@HttpRequest(method=GET)`

When a request is made the request is processed by the underlying `HttpWebServer`, before our `greeting` method is called the parameters are loaded. `@RequestParam` indicates the parameter should be loaded using a query parameter, specifically the `name` parameter. If the parameter is absent the default value is used, which is provided by the `or` attribute in the `@RequestParam` annotation.

## Starting the application
Before you can test your application, you first need to be able to start it. Luckily Hartshorn makes this easy using the `HartshornApplication` class. To start the application, you will need an [activator class](../core/application-bootstrap.md). An activator class is simply a class annotated with `@Activator` and required [service activators](../core/service-activators.md). As we wish to use a REST HTTP server to handle our requests, we will need just one service activator, `@UseHttpServer`.

```java
@Activator // (1)
@UseHttpServer
public class GreetingApplication {

    public static void main(String[] args) { // (2)
        HartshornApplication.create(GreetingApplication.class, args);
    }
}
```

1.  Activators should be located at the base package of your application. This tells Hartshorn to scan for components and services inside that package, allowing it to register the controller.
2.  You can start the Hartshorn application directly from the application entrypoint, as no additional code is required before being able to start Hartshorn.
