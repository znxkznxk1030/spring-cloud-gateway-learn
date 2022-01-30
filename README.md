# Building a Gateway

<https://spring.io/guides/gs/gateway/>

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

## Testing

### Using WireMock

<https://cloud.spring.io/spring-cloud-contract/2.0.x/multi/multi__spring_cloud_contract_wiremock.html>

``` java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT,
    properties = {"httpbin=http://localhost:${wiremock.server.port}"})
@AutoConfigureWireMock(port = 0)
public class ApplicationTest {

  @Autowired
  private WebTestClient webClient;

  @Test
  public void contextLoads() throws Exception {
    //Stubs
    stubFor(get(urlEqualTo("/get"))
        .willReturn(aResponse()
          .withBody("{\"headers\":{\"Hello\":\"World\"}}")
          .withHeader("Content-Type", "application/json")));
    stubFor(get(urlEqualTo("/delay/3"))
      .willReturn(aResponse()
        .withBody("no fallback")
        .withFixedDelay(3000)));

    webClient
      .get().uri("/get")
      .exchange()
      .expectStatus().isOk()
      .expectBody()
      .jsonPath("$.headers.Hello").isEqualTo("World");

    webClient
      .get().uri("/delay/3")
      .header("Host", "www.circuitbreaker.com")
      .exchange()
      .expectStatus().isOk()
      .expectBody()
      .consumeWith(
        response -> assertThat(response.getResponseBody()).isEqualTo("fallback".getBytes()));
  }
}
```