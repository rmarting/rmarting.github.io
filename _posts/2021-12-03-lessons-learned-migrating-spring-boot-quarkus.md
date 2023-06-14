---
layout:      post
title:       ":rocket: Lessons learned migrating Spring Boot to Quarkus"
subtitle:    My thoughts and impressions after the migration from Spring Boot to Quarkus.
description: My thoughts and impressions after the migration from Spring Boot to Quarkus.
date:        2021-12-03 09:15
toc:         true
comments:    true
img:         how-to.avif
tags: 
- How-to
- Quarkus
- Spring Boot
---

This blog post describes a set of lessons learned from my personal experience
migrating Spring Boot applications to Quarkus. The article does not cover all the
topics, approaches, architectures or designs to keep in mind for an enterprise full
migration project, but it includes a set of conclusions from a personal perspective.

Cloud Native Applications, Microservices Architectures, Event-Driven Architectures,
Serverless, … are the most common patterns, designs and topics used by Enterprise,
Start-ups, and Software Companies to design and deploy new applications in this new
Cloud Era (a.k.a [Kubernetes Era](https://www.infoq.com/articles/microservices-post-kubernetes/)).
To build these kinds of new applications exists a list of different technologies,
frameworks and languages (Go, Node.JS, Java, …) however one of the most extended
and used is Spring Boot.

Spring Boot is a well-known framework, with a large community of developers, long
history and friendly for many developers. However Spring Boot has other behaviors
that could not fit well in a Cloud Native environment (resources consumption,
startup time, response time, development lifecycle, …).

[Quarkus](https://quarkus.io/) is the new player in the playground to design
new applications under these paradigms.

Quarkus is a full-stack, Kubernetes-native Java framework made for Java Virtual
Machines and native compilation. Quarkus is crafted from best-of-breed Java
libraries and standards with amazingly fast boot times and incredibly low
memory in container orchestration platforms like Kubernetes.

Quarkus has a clear vision based in:

* [Container First](https://quarkus.io/container-first): Optimized
for low memory usage and fast startup times.
* [Imperative and Reactive](https://quarkus.io/continuum): Designed
with this new world in mind and provides first-class support for these
different paradigms.
* [Developer Joy](https://quarkus.io/developer-joy): Designed to make happy and fun the developer's life.
* [Community and Standards](https://quarkus.io/standards): No need to
learn new technologies, designed on top of proven standards
(Eclipse MicroProfile, JAX-RS, JPA, …)
* [Kube-native](https://quarkus.io/kubernetes-native): Providing tools optimized
for Kubernetes.

**Houston, we have a problem!** Quarkus versus Spring Boot? Quarkus?
Spring Boot? Which one?

In many migration projects (frameworks, application servers, jdk, …) the effort
to adapt the source code to the new platform was one key. Refactoring code implies
a range effort between trivial to epic and it could decline the decision to go
or not to go. Refactoring from Spring Boot to Quarkus could be another one.

This article describes the following migration approaches from Spring Boot to Quarkus:

* Migrating to Quarkus Extensions for Spring Boot
* Refactoring to Standard Libraries and Quarkus Extensions

:rotating_light::rotating_light: **Disclaimer and Spoiler Alert** This article is
not an official migration guide. :rotating_light::rotating_light:

Any kind of migration requires analyzing different things to answer questions
such as why?, who?, when?, how?, where?. These questions are not easy to
analyze and describe in a single article because they involve a large number
of aspects like: processes, people, management, testing, ... but who does not
listen to sentences such as: *Java is dead now*; *XX framework is better for
cloud containers*, *Java consumes a lot of resources*, … Well, **Quarkus changes
many of those sentences**.

This article only wants to focus on some aspects of how to code/refactor
the source code from Spring Boot to Quarkus. It does not cover all the
features or capabilities of both frameworks but it summarizes a set of
lessons learned from my personal experience. Quarkus is a highly active
community so some of these lessons could change in the future (maybe in a few days).

## Application to migrate

This blog post is based on a Spring Boot application with the following
modules or components:

* Spring Boot 2
* REST Endpoints based in Spring Web
* Apache Kafka as messaging system
* Apicurio Service Registry as API schema registry
* Avro schemas

It is a base to start to analyze this migration, where some other common
features are not included (e.g: Persistence in Databases) to reduce the
scope of this migration.

The original code of this application is available [here](https://github.com/rmarting/kafka-clients-sb-sample).

The original application starts in 5 seconds:

```text
2021-12-03 09:33:56.724  INFO 1 --- [           main] com.rmarting.kafka.Application           : Started Application in 5.132 seconds (JVM running for 5.76)
```

## Migrating to Quarkus Extensions for Spring Boot

This approach is focused on reducing the number of changes and reuse as much
code as possible. This approach is available thanks to a set of Quarkus
Extensions for Spring Boot, designed to provide a compatibility layer for
Spring Boot. At the time of writing this article the following Quarkus
Extensions for Spring are available:

* **spring-di**: Compatibility layer for Spring dependency injection.
* **spring-web**: Compatibility layer for Spring Web.
* **spring-boot-properties**: Compatibility layer to set up your Spring Boot
using @ConfigurationProperties annotations.
* **spring-security**: Compatibility layer for Spring Security.
* **spring-cache**: Compatibility layer for Spring Cache annotations.
* **spring-data-jpa**: Compatibility layer for Spring Data JPA repositories.
* **spring-scheduled**: Compatibility layer for Spring Scheduled.
* **spring-cloud-config-client**: Compatibility layer to read
configuration properties at runtime from the Spring Cloud Config Server.

**NOTE**: Some extensions are considered preview, so backward compatibility
and presence in the ecosystem is not guaranteed.

The main list of changes done to migrate the application is:

* Quarkus requires JDK 11.
* `spring-di`, `spring-web` and `spring-boot-properties` extensions are
basically *mandatory* for any Spring Boot applications. We could say that
they are the base for any migration to Quarkus Spring.
* These extensions provide the base features of Spring Boot such
as: Dependency Injection, Web, Configuration.
* Although the compatibility layer supports most of the Spring DI
capabilities, some arcane features may not be supported.
* Spring Web Annotations could be maintained exactly equals. 
* Swagger annotations must be refactored to use MicroProfile OpenAPI.
This refactor basically needs to use classes from
`org.eclipse.microprofile.openapi.annotations` package.
* OpenAPI and Swagger capabilities are provided now by the
`quarkus-smallrye-openapi` extension, not needed by other
OpenAPI or Swagger dependencies.
* Health checks (actuators provided by `spring-boot-starter-actuator`)
are now provided by `quarkus-smallrye-health` extensions. It requires
new liveness and readiness proves in your deployment in Kubernetes.
* Integration with Kafka using the Kafka Producer and Consumer API
(provided by the Kafka Clients) only requires the use of 
`quarkus-kafka-client` extension.
* Spring Kafka has not an equivalent compatible Spring extension,
however Quarkus provides `quarkus-smallrye-reactive-messaging-kafka` extension
with a set of new annotations to consume, produce or data streaming
with Apache Kafka. It means a small set of changes.

The result of this migration is available [here](https://github.com/rmarting/kafka-clients-sb-sample/tree/feature/quarkus-edition).

This new application starts in less of 2 seconds:

```text
Dec 03, 2021 9:51:17 AM io.quarkus.bootstrap.runner.Timing printStartupTime
INFO: kafka-clients-sb-sample 3.0.0-SNAPSHOT on JVM (powered by Quarkus 1.13.7.Final) started in 1.887s. Listening on: http://0.0.0.0:8181
```

## Refactoring to Standard Libraries and Quarkus Extensions

This approach implies a change in the mindset of your application, aligning
with standards and refactoring your code. However you will get a
full-compliant Quarkus application and then you are able to use all its
empowerment.

For this approach I need to map the Spring Boot features with the
right Quarkus Extensions:

| Spring Boot Feature | Quarkus Extension |
|---------------------|-------------------|
| spring-di | [Quarkus CDI](https://quarkus.io/guides/cdi) |
| spring-boot-starter-web | [JAX-RS Services](https://quarkus.io/guides/rest-json) |
| spring-boot-starter-actuator | [MicroProfile Health](https://quarkus.io/guides/smallrye-health) |
| springdoc-openapi-ui | [OpenAPI and Swagger UI](https://quarkus.io/guides/openapi-swaggerui) |
| spring-kafka | [Kafka with Reactive Messaging](https://quarkus.io/guides/kafka) |

The main list of changes done to migrate the application is:

* Quarkus requires JDK 11.
* Refactor `@Service`, `@Component`, `@Autowired` Spring annotations
to `@Singleton`, `@ApplicationScoped`, `@Inject` CDI annotations.
* Refactor `@Configuration`, `@Value` Spring annotations to
`@ApplicationScoped`, `@ConfigProperty` Quarkus annotations
* Refactor Spring Web Annotations to JAX-RS Annotations. This guide
includes a [conversion table](https://quarkus.io/guides/spring-web#conversion-table).
* Swagger annotations must be refactored to use MicroProfile OpenAPI. This
refactor basically needs to use classes from
`org.eclipse.microprofile.openapi.annotations` package.
* OpenAPI and Swagger capabilities are provided now by 
`quarkus-smallrye-openapi` extension, not needed by other
OpenAPI or Swagger dependencies.
* Health checks (actuators provided by `spring-boot-starter-actuator`) are
now provided by `quarkus-smallrye-health` extension. It requires new
liveness and readiness proves in your deployment in Kubernetes.
* Integration with Kafka using the Kafka Producer and Consumer API
(provided by the Kafka Clients) only requires the use of 
`quarkus-kafka-client` extension.
* Spring Kafka has not an equivalent compatible Spring extension,
however Quarkus provides `quarkus-smallrye-reactive-messaging-kafka` extension
with a set of new annotations to consume, produce or data streaming
with Apache Kafka. It means a small set of changes.

The result of this refactoring is available [here](https://github.com/rmarting/kafka-clients-quarkus-sample).

This application starts in 1.4 seconds:

```text
Dec 03, 2021 10:21:17 AM io.quarkus.bootstrap.runner.Timing printStartupTime
INFO: kafka-clients-sb-sample 3.0.0-SNAPSHOT on JVM (powered by Quarkus 1.13.7.Final) started in 1.387s. Listening on: http://0.0.0.0:8181
```

## Lessons Learned

Both migration experiences could be summarized in the next lessons learned:

* Migration to Quarkus has been feasible with less effort migration.
Both approaches did not require a full investment or hard work.
* Quarkus Extensions for Spring Boot are enough for most common modules
(`di`, `web`, `jpa`, `security`, …) but maybe not cover yours
(`jta`, `web-services`, complex injection references, …). Analyzing the
application is **mandatory** to get the gap.
* Quarkus Extensions for Spring Boot could not cover all Spring
features, then a refactor to Quarkus could be needed
(Health Endpoints, OpenAPI, Swagger UI, Messaging Integration).
However, the effort to refactor it was not so hard.
* Refactoring to Quarkus involves moving your code to standard
libraries (that hopefully you already know like for instance JAX-RS), so
the learning curve to start with Quarkus is minimal.
* Refactor to Quarkus will give you the full-power of Quarkus and
its extensions.
* Bugs and issues could appear in both migrations. Quarkus is stable
and it is growing up so fast, resolving them and adding new features.
You can check it in the [releases page](https://github.com/quarkusio/quarkus/releases).
* Full Quarkus migrated application was the faster one after
completing the refactoring, as you can see in the following chart.
It is not a complete performance test, but it might give you an
idea about the performance capabilities of Quarkus:

| Implementation | Startup |
|---|---|
| Spring Boot | 5 seconds |
| Quarkus extensions for Spring | 1.7 seconds :rocket: |
| Quarkus | 1.4 seconds :rocket: |
| Quarkus Native | 0.042 seconds :rocket::rocket: |

There is a list of other components not covered (e.g: testing, JPA,
Cloud integration, security, …) in this blog post because it will
extend the scope and length of this article. Quarkus is a highly
active community where every day new features, issues and extensions are
moving so some of these lessons learned could be covered, resolved or fixed soon.

## Getting starting my migration

Migrating Spring Boot to Quarkus requires an effort to identify
the best approach from the current state (AS-IS) to the final
state (TO-BE) . There is not a silver bullet to migrate applications
but there are some tools and references that could help:

* [Red Hat Migration Toolkit for Applications](https://developers.redhat.com/products/mta/overview):
This tool could analyze your code to identify the main migration issues.
The latest version includes a set of rules to check your code and identify
the main issues to migrate to Quarkus.
* [Quarkus Guides](https://quarkus.io/guides/) are a great resource
for getting started in Quarkus.
* [Quarkus QuickStarts](https://github.com/quarkusio/quarkus-quickstarts) is
a large repository of code with many samples.
* [Quarkus Blog](https://quarkus.io/blog/).
* [Migrating SpringBoot PetClinic REST to Quarkus](https://dzone.com/articles/migrating-a-spring-boot-application-to-quarkus-cha) by
Jonathan Vila as another migration reference from Spring Boot.
* [Migrating a Spring Boot microservices application to Quarkus](https://developers.redhat.com/blog/2020/04/10/migrating-a-spring-boot-microservices-application-to-quarkus)
* [Migrating Spring Boot tests to Quarkus](https://developers.redhat.com/blog/2020/07/17/migrating-spring-boot-tests-to-quarkus)

:tada: Enjoy your journey to Quarkus. :tada:
