---
layout:     post
title:      ":zap: Integrating Quarkus with Apicurio Service Registry"
date:       2020-12-18 09:15
toc:        true
comments:   true
img:        how-to.jpg
tags: 
- How-to
- Quarkus
- Apicurio
- Strimzi
- Operators
- OpenShift
- tutorial
---

Step by step most of the new cloud native applications and microservices designs are based in
[event-driven architecture (EDA)](https://developers.redhat.com/topics/event-driven) to respond to real-time information
based on sending and receiving of information about individual events. This kind of architecture is based on asynchronous
non-blocking communication between event producers and consumers through a event streaming backbone, such
as Apache Kafka running on top of Kubernetes. In these scenarios, where a large number of different events are
managed, is very important to define a governance model where each event could be defined as an API to allow producers
and consumers to produce and consume checked and validated events. A Service Registry will help us.

From my field experience with many projects I found that the most typical landscape is based in the most well-know next components:

* [Strimzi](https://strimzi.io/) to deploy Apache Kafka clusters as streaming backbone.
* [Apicurio Service Registry](https://www.apicur.io/) as datastore for events API.
* [OpenShift Container Platform](https://www.openshift.com/) to deploy and run the different components.
* [Quarkus](https://quarkus.io/) as framework to develop client applications.
* [Avro](https://avro.apache.org/) as data serialization system to declare schemas as events API.

This article describes how easy is to integrate your Quarkus applications with Apicurio Service Registry.

## :robot: Apicurio Service Registry

Service Registry is a datastore for sharing standard event schemas and API designs across API and event-driven architectures.
Service Registry decouples the structure of your data from your client applications, and to share and manage your data types
and API descriptions at runtime. Decouples your data structure from your client applications reduces costs by decreasing
overall message size, creates efficiencies by increasing consistent reuse of schemas and API designs across your organization.

Some of the most common uses cases where Service Registry helps us are:

* Client applications can dynamically push or pull the latest schema updates to or from Service Registry at
runtime without needing to redeploy.
* Developer teams can query the registry for existing schemas required for services already deployed in production.
* Developer teams can register new schemas required for new services in development or rolling to production.
* Store schemas used to serialize and deserialize messages, which can then be referenced from your client
applications to ensure that the messages that they send and receive are compatible with those schemas.

Apicurio is a open source project that provides a Service Registry ready to be involved in this scenario with the
following main features:

* Support for multiple payload formats for standard event schemas and API specifications.
* Pluggable storage options including AMQ Streams, embedded Infinispan, or PostgreSQL database.
* Registry content management using a web console, REST API command, Maven plug-in, or Java client.
* Rules for content validation and version compatibility to govern how registry content evolves over time.
* Full Apache Kafka schema registry support, including integration with Kafka Connect for external systems.
* Client serializer/deserializer (Serdes) to validate Kafka and other message types at runtime.
* Cloud-native Quarkus Java runtime for low memory footprint and fast deployment times.
* Compatibility with existing Confluent schema registry client applications.
* Operator-based installation of Service Registry on OpenShift.

## :computer: Client Applications Workflow

The typical workflow when we introduce a Service Registry in our architecture is:

* Declare the event schema using some of the most data formats like Apache Avro, JSON Schema, Google Protocol Buffers,
OpenAPI, AsyncAPI, GraphQL, Kafka Connect schemas, WSDL or XML schemas (xsd).
* Register the schema as artifact in Service Registry through the Service Registry UI, API REST, Maven PlugIn or
Java clients. From there client applications can then use that schema to validate that messages conform to the correct
data structure at runtime.
* Kafka Producer applications use a serializer to encode messages that conform to a specific event schema.
* Kafka Consumer applications then use a deserializer to validate that messages have been serialized using that
correct schema, based on a specific schema ID.

This workflow ensures consistent schema use and helps to prevent data errors at runtime.

## :page_facing_up: Avro Schemas into Service Registry

Avro provides a [JSON schema specification](https://avro.apache.org/docs/current/spec.html#schemas) to declare a
large variety of data structures, such as our simple example:

```json
{
  "name": "Message",
  "namespace": "io.jromanmartin.kafka.schema.avro",
  "type": "record",
  "doc": "Schema for a Message.",
  "fields": [
    {
      "name": "timestamp",
      "type": "long",
      "doc": "Message timestamp."
    },
    {
      "name": "content",
      "type": "string",
      "doc": "Message content."
    }
  ]
}
```

This schema will define a simple message event.

Avro also provides a [Maven Plugin](https://avro.apache.org/docs/current/gettingstartedjava.html) to autogenerate
Java classes based in the schema definitions (```.avsc``` files)

Now we could publish it into Service Registry to be used in runtime for our client applications. Apicurio Maven Plugin
is an easy way to publish the schemas into the Service Registry with a simple definition in our ```pom.xml``` file.

```xml
<plugin>
   <groupId>io.apicurio</groupId>
   <artifactId>apicurio-registry-maven-plugin</artifactId>
   <version>${apicurio.version}</version>
   <executions>
      <execution>
         <phase>generate-sources</phase>
         <goals>
            <goal>register</goal>
         </goals>
         <configuration>
            <registryUrl>${apicurio.registry.url}</registryUrl>
            <artifactType>AVRO</artifactType>
            <artifacts>
               <!-- Schema definition for TopicIdStrategy strategy -->
               <messages-value>${project.basedir}/src/main/resources/schemas/message.avsc</messages-value>
            </artifacts>
         </configuration>
      </execution>
   </executions>
</plugin>
```

Using Apicurio Maven Plugin in the our application Maven Lifecycle could help us to define or extend our ALM
(also our CI/CD pipelines) to publish or update the schemas every time we released new versions of them. It
is not an objective of this article, but something that you could analyze more.

As soon we published our schema into Service Registry we could manage from the UI as:

{:refdef: style="text-align: center;"}
[![](/images/apicurio-registry/apicurio-artifact-details.png "Apicurio Service Registry Artifact Details")]({{site.url}}/images/apicurio-registry/apicurio-artifact-details.png)
{: refdef}

## :rocket: Quarkus, Apache Kafka and Service Registry

Quarkus provides a set of dependencies to allow in our application to produce and consume messages to and
from Apache Kafka. It is very straight forward to use it onces we add the dependency in our ```pom.xml``` file:

```xml
<!-- Kafka Clients -->
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-kafka-client</artifactId>
</dependency>
<!-- Reactive Messaging -->
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-reactive-messaging-kafka</artifactId>
</dependency>
```

Connect your application with the Apache Kafka cluster is so easy with the following property
in your ```application.properties``` file:

```text
kafka.bootstrap.servers = my-kafka-kafka-bootstrap:9092
```

Apicurio Service Registry provides Kafka client serializer/deserializer for Kafka producer and consumer
applications. To add them into our application, you must add the next dependency:

```xml
<!-- Apicurio Serializer/Deserializer -->
<dependency>
    <groupId>io.apicurio</groupId>
    <artifactId>apicurio-registry-utils-serde</artifactId>
    <version>${apicurio.version}</version>
</dependency>
```

### :inbox_tray: Producing Messages from Quarkus

Quarkus provides a set of properties and beans to declare Kafka Producers to send
messages (in our case Avro-schema instances) to Apache Kafka. The most important properties to set up are:

* ```key.serializer```: Identifies the serializer class to serialize the key of the Kafka record.
* ```value.serializer```: Identifies the serializer class to serialize the value of the Kafka record.

Here we have to add some specific values in these properties to allow the serialization process using Avro schemas
registered in the Service Registry. Basically we need to identify the following concepts:

* Serializer class to use Avro schemas, this class is provider by the Apicurio SerDe class: ```io.apicurio.registry.utils.serde.AvroKafkaSerializer```
* Apicurio Service registry endpoint to validate schemas: ```apicurio.registry.url```
* Apicurio Service strategy to look up the schema definition: ```apicurio.registry.artifact-id```

So a sample definition for a producer bean to send messages could be similar to:

```java
    @Produces
    @RequestScoped
    public Producer<String, Message> createProducer() {
        Properties props = new Properties();

        // Kafka Bootstrap
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, kafkaBrokers);

        // Security
        props.put(AdminClientConfig.SECURITY_PROTOCOL_CONFIG, kafkaSecurityProtocol);
        props.put(SaslConfigs.SASL_MECHANISM, "SCRAM-SHA-512");
        props.put(SaslConfigs.SASL_JAAS_CONFIG,
                "org.apache.kafka.common.security.scram.ScramLoginModule required username=\"" + kafkaUser
                        + "\" password=\"" + kafkaPassword + "\";");

        // Producer Client
        props.putIfAbsent(ProducerConfig.CLIENT_ID_CONFIG, producerClientId + "-" + getHostname());

        // Serializer for Keys and Values
        props.putIfAbsent(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.putIfAbsent(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, AvroKafkaSerializer.class.getName());

        // Service Registry
        props.putIfAbsent(AbstractKafkaSerDe.REGISTRY_URL_CONFIG_PARAM, serviceRegistryUrl);
        // Topic Id Strategy (schema = topicName-(key|value)) - Default Strategy
        props.putIfAbsent(AbstractKafkaSerializer.REGISTRY_ARTIFACT_ID_STRATEGY_CONFIG_PARAM, TopicIdStrategy.class.getName());

        // Acknowledgement
        props.putIfAbsent(ProducerConfig.ACKS_CONFIG, acks);

        return new KafkaProducer<>(props);
    }
```

And you could finally send Messages (storing the artifact ID from Service Registry) to Apache Kafka:

```java
    @Inject
    Producer<String, Message> producer;

    private MessageDTO publishRawMessage(final @NotEmpty String topicName,
                                         final @NotNull MessageDTO messageDTO,
                                         final boolean async) {
        // ... 
        RecordMetadata metadata = producer.send(record).get();
        // ....
    }
```

The message will be serialized adding the global ID associated to the schema used for this record. That
global ID will be very important to consume later the message by the Kafka Consumer applications.

**NOTE:** This approach uses KafkaProducer API, however Quarkus includes the ```Emitter``` class
to send messages easily. I developed my sample with the first approach to check that Kafka API is still
valid using Quarkus.

### :outbox_tray: Consuming Messages from Quarkus

Quarkus also provides a set of properties and beans to declare Kafka Consumers to consume messages (in our case Avro-schema instances)
from the Apache Kafka cluster. The most important properties to set up are:

* ```key.deserializer```: Identifies the deserializer class to deserialize the key of the Kafka record.
* ```value.deserializer```: Identifies the deserializer class to deserialize the value of the Kafka record.

Here we have to add some specific values in these properties to allow the deserialization process using
Avro schemas registered in the Service Registry. Basically we need to identify the following concepts:

* Deserializer class to use Avro schemas, this class is provider by the Apicurio SerDe class: ```io.apicurio.registry.utils.serde.AvroKafkaDeserializer```
* Apicurio Service registry endpoint to get valid schemas: ```apicurio.registry.url```

So a sample configuration for a consumer template could be similar to:

```text
# Configure the Kafka source (we read from it)
mp.messaging.incoming.messages.connector=smallrye-kafka
mp.messaging.incoming.messages.group.id=${app.consumer.groupId}-mp-incoming-channel
mp.messaging.incoming.messages.topic=messages.${app.bg.mode}
mp.messaging.incoming.messages.key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
mp.messaging.incoming.messages.value.deserializer=io.apicurio.registry.utils.serde.AvroKafkaDeserializer
mp.messaging.incoming.messages.properties.partition.assignment.strategy=org.apache.kafka.clients.consumer.RoundRobinAssignor
mp.messaging.incoming.messages.apicurio.registry.url=${apicurio.registry.url}
mp.messaging.incoming.messages.apicurio.registry.avro-datum-provider=io.apicurio.registry.utils.serde.avro.ReflectAvroDatumProvider
```

You could declare a listener to consume messages (based in our Message schema) as:

```java
    @Incoming("messages")
    public CompletionStage handleMessages(Message message) {
        IncomingKafkaRecord<String, io.jromanmartin.kafka.schema.avro.Message> incomingKafkaRecord =
                (IncomingKafkaRecord<String, io.jromanmartin.kafka.schema.avro.Message>) message.unwrap(IncomingKafkaRecord.class);

        LOGGER.info("Received record from Topic-Partition '{}-{}' with Offset '{}' -> Key: '{}' - Value '{}'",
                incomingKafkaRecord.getTopic(),
                incomingKafkaRecord.getPartition(),
                incomingKafkaRecord.getOffset(),
                incomingKafkaRecord.getKey(),
                incomingKafkaRecord.getPayload());

        // Commit message
        return message.ack();
    }
```

The schema is retrieved by the deserializer using the global ID written into the message being consumed from the Service Registry. **Done!**

## :bookmark_tabs: Summary

Your Quarkus applications integrate easily with the Service Registry and Apache Kafka to build your
event-driven architecture. Thanks all these components you could get the following main benefits:

* Ensure consistent schema use between your client applications.
* Help to prevent data errors at runtime.
* Define a governance model in your data schemas (versions, rules, validations).
* Easy integration with client applications and components.

You could invest and analyze more to adapt or build your event-drive architecture with these components in the following reference links.

* [Getting started with Service Registry](https://www.apicur.io/registry/docs/apicurio-registry/1.3.3.Final/index.html)
* [First look at the new Apicurio Registry UI and Operator](https://developers.redhat.com/blog/2020/06/11/first-look-at-the-new-apicurio-registry-ui-and-operator/)
* [How to Use Kafka, Schema Registry and Avro with Quarkus](https://quarkus.io/blog/kafka-avro/)
* [Using Avro in a native executable](https://quarkus.io/blog/avro-native/)

## :microscope: Show me the code

Everything seems great and cool, but you want to see how it is really working ... then 
this [GitHub repository](https://github.com/rmarting/kafka-clients-quarkus-sample) is your reference.

Enjoying API eventing :smiley:!!!

## Bonus Track

Quarkus 1.10.5.Final includes the native compilation of the Avro schemas capability. This command
will compile as native my sample application:

```bash
❯ mvn package -Dquarkus.native.container-build=true -Dquarkus.native.container-runtime=podman -Pnative
```

But this is other story to write in other post! :sunglasses:
