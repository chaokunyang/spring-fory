# Spring Fory

[![fory-json-spring Maven Central](https://img.shields.io/maven-central/v/io.github.chaokunyang/fory-json-spring?style=for-the-badge&label=fory-json-spring)](https://central.sonatype.com/artifact/io.github.chaokunyang/fory-json-spring)
[![fory-json-spring6 Maven Central](https://img.shields.io/maven-central/v/io.github.chaokunyang/fory-json-spring6?style=for-the-badge&label=fory-json-spring6)](https://central.sonatype.com/artifact/io.github.chaokunyang/fory-json-spring6)
[![fory-json-spring-boot-starter Maven Central](https://img.shields.io/maven-central/v/io.github.chaokunyang/fory-json-spring-boot-starter?style=for-the-badge&label=fory-json-spring-boot-starter)](https://central.sonatype.com/artifact/io.github.chaokunyang/fory-json-spring-boot-starter)
[![fory-json-spring-boot3-starter Maven Central](https://img.shields.io/maven-central/v/io.github.chaokunyang/fory-json-spring-boot3-starter?style=for-the-badge&label=fory-json-spring-boot3-starter)](https://central.sonatype.com/artifact/io.github.chaokunyang/fory-json-spring-boot3-starter)

Spring MVC message converters and Spring WebFlux codecs for
[Apache Fory JSON](https://fory.apache.org/), with auto-configuration starters for Spring Boot.

Spring Fory requires Java 17 or newer. Choose one complete version line and keep every Spring Fory
artifact on the same version:

| Spring Framework | Direct adapter | Spring Boot | Starter |
| --- | --- | --- | --- |
| 7.x | `io.github.chaokunyang:fory-json-spring` | 4.x | `io.github.chaokunyang:fory-json-spring-boot-starter` |
| 6.2.x | `io.github.chaokunyang:fory-json-spring6` | 3.5.x | `io.github.chaokunyang:fory-json-spring-boot3-starter` |

## Installation

The artifacts are available from Maven Central, so no additional repository configuration is
required.

For Spring Boot 4, add:

```xml
<dependency>
  <groupId>io.github.chaokunyang</groupId>
  <artifactId>fory-json-spring-boot-starter</artifactId>
  <version>1.0.0</version>
</dependency>
```

For Spring Boot 3.5, add:

```xml
<dependency>
  <groupId>io.github.chaokunyang</groupId>
  <artifactId>fory-json-spring-boot3-starter</artifactId>
  <version>1.0.0</version>
</dependency>
```

A starter includes its matching Spring adapter. Do not add the adapter separately or combine the
two version lines. Keep the application's existing Spring MVC or Spring WebFlux starter.

For an application using Spring Framework directly, use the adapter from the compatibility table
instead of a Boot starter. For example, with Spring Framework 7:

```xml
<dependency>
  <groupId>io.github.chaokunyang</groupId>
  <artifactId>fory-json-spring</artifactId>
  <version>1.0.0</version>
</dependency>
```

Use `fory-json-spring6` with Spring Framework 6.2.

## Spring Boot usage

The starter detects Spring MVC or Spring WebFlux and registers the matching Fory JSON integration.
It also creates one thread-safe `ForyJson` bean unless the application provides one.

### Spring MVC

A normal controller can read and write DTOs through Fory JSON:

```java
import org.apache.fory.json.annotation.JsonProperty;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/users")
public final class UserController {
  @PostMapping(
      consumes = MediaType.APPLICATION_JSON_VALUE,
      produces = MediaType.APPLICATION_JSON_VALUE)
  public User echo(@RequestBody User user) {
    return user;
  }

  public record User(long id, @JsonProperty("display_name") String name) {}
}
```

Sending `{"id":7,"display_name":"Alice"}` returns the same JSON properties. Declared generic
types such as `List<User>` and `Map<String, User>` retain their type arguments during conversion.

### Spring WebFlux

The WebFlux codecs support single values, JSON arrays, and newline-delimited JSON (NDJSON):

```java
import org.apache.fory.json.annotation.JsonProperty;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
public final class ReactiveUserController {
  @PostMapping(path = "/users/one", consumes = MediaType.APPLICATION_JSON_VALUE)
  public Mono<User> echo(@RequestBody Mono<User> user) {
    return user;
  }

  @GetMapping(path = "/users/array", produces = MediaType.APPLICATION_JSON_VALUE)
  public Flux<User> array() {
    return Flux.just(new User(7, "Alice"), new User(8, "Bob"));
  }

  @GetMapping(path = "/users/stream", produces = MediaType.APPLICATION_NDJSON_VALUE)
  public Flux<User> stream() {
    return Flux.just(new User(7, "Alice"), new User(8, "Bob"));
  }

  public record User(long id, @JsonProperty("display_name") String name) {}
}
```

`Mono<User>` with `application/json` reads or writes one value. `Flux<User>` with
`application/json` uses a JSON array, while `application/x-ndjson` writes one JSON value per line.

## Direct Spring Framework setup

Applications without Spring Boot create a `ForyJson` instance and register the matching adapter.

### Spring MVC

Spring Framework 7 uses the server message-converter builder:

```java
import io.github.chaokunyang.springfory.ForyJsonHttpMessageConverter;
import org.apache.fory.json.ForyJson;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverters;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class ForyMvcConfiguration implements WebMvcConfigurer {
  private final ForyJson json = ForyJson.builder().build();

  @Override
  public void configureMessageConverters(HttpMessageConverters.ServerBuilder builder) {
    builder.withJsonConverter(new ForyJsonHttpMessageConverter(json));
  }
}
```

Spring Framework 6.2 adds the converter before the other configured JSON converters:

```java
import io.github.chaokunyang.springfory.ForyJsonHttpMessageConverter;
import java.util.List;
import org.apache.fory.json.ForyJson;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class ForyMvcConfiguration implements WebMvcConfigurer {
  private final ForyJson json = ForyJson.builder().build();

  @Override
  public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters.add(0, new ForyJsonHttpMessageConverter(json));
  }
}
```

### Spring WebFlux

Spring Framework 7 replaces the default JSON encoder and decoder:

```java
import io.github.chaokunyang.springfory.ForyJsonDecoder;
import io.github.chaokunyang.springfory.ForyJsonEncoder;
import org.apache.fory.json.ForyJson;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.codec.ServerCodecConfigurer;
import org.springframework.web.reactive.config.WebFluxConfigurer;

@Configuration
public class ForyWebFluxConfiguration implements WebFluxConfigurer {
  private final ForyJson json = ForyJson.builder().build();

  @Override
  public void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
    configurer.defaultCodecs().jacksonJsonEncoder(new ForyJsonEncoder(json));
    configurer.defaultCodecs().jacksonJsonDecoder(new ForyJsonDecoder(json));
  }
}
```

For Spring Framework 6.2, use the corresponding Jackson 2 codec slots:

```java
configurer.defaultCodecs().jackson2JsonEncoder(new ForyJsonEncoder(json));
configurer.defaultCodecs().jackson2JsonDecoder(new ForyJsonDecoder(json));
```

## Customize Fory JSON

Define a `ForyJson` bean to change Fory JSON settings. The starter will use it instead of creating a
default instance:

```java
import org.apache.fory.json.ForyJson;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class JsonConfiguration {
  @Bean
  ForyJson foryJson() {
    return ForyJson.builder().writeNullFields(true).build();
  }
}
```

When the starter creates the default instance, it discovers and installs every `ForyJsonModule`
bean in Spring order. Fory annotations and installed modules control JSON mapping; Jackson
annotations and `spring.jackson.*` properties do not configure Fory JSON.

## Configure input limits

Both starters limit a complete Spring MVC request body with:

```properties
fory.json.max-input-bytes=67108864
```

The default is 64 MiB and the value must be positive.

For Spring WebFlux, use the standard property for the application's Spring Boot line:

```properties
# Spring Boot 4
spring.http.codecs.max-in-memory-size=8MB

# Spring Boot 3.5
spring.codec.max-in-memory-size=8MB
```

## Build

```shell
./mvnw verify
```
