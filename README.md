# Spring Fory

Spring MVC message converters and Spring WebFlux codecs for [Apache Fory JSON](https://fory.apache.org/), plus auto-configuration starters for Spring Boot.

The project requires Java 17 or newer and provides two compatibility lines:

| Spring Framework | Direct adapter | Spring Boot | Starter |
| --- | --- | --- | --- |
| 7.x | `fory-json-spring` | 4.x | `fory-json-spring-boot-starter` |
| 6.2.x | `fory-json-spring6` | 3.5.x | `fory-json-spring-boot3-starter` |

See the [Spring and Spring Boot guide](docs/spring-boot.md) for configuration and usage examples.

## Build

```shell
./mvnw verify
```
