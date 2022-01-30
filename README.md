# Building a Gateway

## Using Spring Cloud CircuitBreaker

```xml
<!-- circuitbreaker -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

``` java
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route(p -> p
            .path("/get")
            .filters(f -> f.addRequestHeader("Hello", "World"))
            .uri("http://httpbin.org:80"))
        .route(p -> p
            .host("*.circuitbreaker.com")
            .filters(f -> f.circuitBreaker(config -> config.setName("mycmd")))
            .uri("http://httpbin.org:80")).
        build();
}
```