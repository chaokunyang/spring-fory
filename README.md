# Spring Fory

Spring MVC message converters and Spring WebFlux codecs for [Apache Fory JSON](https://fory.apache.org/), plus auto-configuration starters for Spring Boot.

The project requires Java 17 or newer and provides two compatibility lines:

| Spring Framework | Direct adapter | Spring Boot | Starter |
| --- | --- | --- | --- |
| 7.x | `fory-json-spring` | 4.x | `fory-json-spring-boot-starter` |
| 6.2.x | `fory-json-spring6` | 3.5.x | `fory-json-spring-boot3-starter` |

See the [Spring and Spring Boot guide](docs/spring-boot.md) for configuration and usage examples.

For Spring Boot 4, add the starter published through GitHub Packages:

```xml
<dependency>
  <groupId>io.github.chaokunyang</groupId>
  <artifactId>fory-json-spring-boot-starter</artifactId>
  <version>1.0.0</version>
</dependency>
```

## Build

```shell
./mvnw verify
```

## Release

Push a semantic-version tag to publish all modules to GitHub Packages. The release workflow derives
the Maven version from the tag, verifies the complete reactor, and deploys the binary, source, and
Javadoc artifacts.

```shell
git tag v1.0.0
git push origin v1.0.0
```
