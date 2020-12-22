---
layout:     post
title:      "My own Apache Kafka Cheat Sheet"
toc:        true
comments:   true
permalink:  /cheat-sheets/apache-kafka
tags:
- How-to
- Cheat Sheet
- Apache Kafka
- tutorial
---

[Apache Kafka](https://kafka.apache.org/) is an open-source distributed event streaming platform used
for high-performance data pipelines, streaming analytics, data integration, and mission-critical applications.

This cheat sheet includes the most common commands to install, deploy, administrate or
operate an Apache Kafka cluster.

**NOTE**: Apache Kafka is the upstream project of
[Red Hat AMQ Streams](https://access.redhat.com/products/red-hat-amq#streams), so these commands are also
valid for this product.

Enjoy it !!! :cool: :sunglasses:

## Topics Management

Listing topics:

```bash
❯ ./bin/kafka-topics.sh --list \
  --bootstrap-server localhost:9092 \
  --exclude-internal
```

Creating topics:

```bash
❯ ./bin/kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic sample-topic --partitions 9 --replication-factor 3
```

Describing topics:

```bash
❯ ./bin/kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --topic sample-topic
```

Deleting topics:

```bash
❯ ./bin/kafka-topics.sh --delete \
  --bootstrap-server localhost:9092 \
  --topic sample-topic	
```

Altering topics:

```bash
❯ ./bin/kafka-configs.sh --alter \
  --zookeeper localhost:2181 \
  --entity-type topics --entity-name sample-topic \
  --add-config min.insync.replicas=2
```

## Topics Reassignment

Generate topics reassignment json file:

```bash
❯ ./bin/kafka-reassign-partitions.sh --generate \
  --bootstrap-server=localhost:9092 --zookeeper=localhost:2181 \
  --broker-list 0,1,2 --topics-to-move-json-file ./reassignment-topics.json
```

Execute topics reassignment:

```bash
❯ ./bin/kafka-reassign-partitions.sh --execute \
  --bootstrap-server=localhost:9092 --zookeeper=localhost:2181 \
  --broker-list 0,1,2 --topics-to-move-json-file ./reassignment-topics.json
```

Verify topics reassignment:

```bash
❯ ./bin/kafka-reassign-partitions.sh --verify \
  --bootstrap-server=localhost:9092 --zookeeper=localhost:2181 \
  --reassignment-json-file ./new-reassignment-topics.json
```

## Sample Producers

* Console Producer: This tool helps to read data from standard input and publish it to Kafka.

```bash
❯ ./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic sample-topic
```                 

* Performance Producer: This tool is used to verify the producer performance.

```bash
❯ ./bin/kafka-producer-perf-test.sh --topic sample-topic \
  --num-records 1000000 --record-size 1024 \
  --throughput -1 \
  --producer.config ./config/producer.properties --print-metrics
```

The ```producer.properties``` file will be similar to:

```text
bootstrap.servers=localhost:9092
compression.type=none
```

In case the Apache Kafka requires SASL authentication with SCRAM-SHA-512, the following properties must be declared:

```text
# SASL by plain connection
security.protocol=SASL_PLAINTEXT
# SASL by secured connection (SSL)
#security.protocol=SASL_SSL
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=user password=passw0rd;
```

**NOTE:** The latest semicolon in the ```sasl.jaas.config``` property is mandatory.

Sample command:

```bash
❯ ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 \
  --topic sample-topic \
  --producer-property security.protocol=SASL_PLAINTEXT \
  --producer-property sasl.mechanism=SCRAM-SHA-512 \
  --producer-property "sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=user password=passw0rd;"
```

## Sample Consumers

* Console Consumer: This tool helps to read data from Kafka and show in console.

```bash
❯ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic sample-topic --group group01 --from-beginning
```

* Performance Consumer: This tool helps in performance test for consumer data.

```bash
❯ ./bin/kafka-consumer-perf-test.sh --broker-list localhost:9092 \
  --topic sample-topic \
  --group perf-consumer-01 --messages 1000000 --timeout 60000 \
  --reporting-interval 1000 --show-detailed-stats --threads 10 \
  --consumer.config ./config/consumer.properties
```

The ```consumer.properties``` file will be similar to:

```text
bootstrap.servers=localhost:9092
group.id=consumer-group
```

In case the Apache Kafka requires SASL authentication with SCRAM-SHA-512, the following properties must be declared:

```text
# SASL by plain connection
security.protocol=SASL_PLAINTEXT
# SASL by secured connection (SSL)
#security.protocol=SASL_SSL
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=user password=passw0rd;
```

Sample command:

```bash
❯ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic sample-topic \
  --from-beginning \
  --max-messages 1 \
  --consumer-property security.protocol=SASL_PLAINTEXT \
  --consumer-property sasl.mechanism=SCRAM-SHA-512 \
  --consumer-property "sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=user password=passw0rd;"
```

## Consumer Group Management

* Listing Consumer Groups:

```bash
❯ ./bin/kafka-consumer-groups.sh --list \
  --bootstrap-server localhost:9092 \
  --command-config ./config/consumer.properties
```

* Describing a Consumer Group: This command gives a lot of information about a consumer group:

```bash
❯ ./bin/kafka-consumer-groups.sh --describe --group sample-consumer \
  --bootstrap-server localhost:9092 \
  --command-config ./config/consumer.properties
```

### Reset Consumer Group

From time to time you need to reset the consumer groups to advance or rewind messages. The following
commands help you to do it:

* Advance to the latest offset of a consumer group:

```bash
❯ ./bin/kafka-consumer-groups.sh --reset-offsets --to-latest --execute \
  --topic sample-topic \ 
  --group sample-consumer \
  --bootstrap-server localhost:9092 \
  --command-config ./config/consumer.properties  
```

* Rewind to the beginning:

```bash
❯ ./bin/kafka-consumer-groups.sh --reset-offsets --to-earliest --execute \
  --topic sample-topic \ 
  --group sample-consumer \
  --bootstrap-server localhost:9092 \
  --command-config ./config/consumer.properties  
```

* Move to an specific offset (e.g: 100):

```bash
❯ ./bin/kafka-consumer-groups.sh --reset-offsets --to-offset 100 --execute \
  --topic sample-topic \ 
  --group sample-consumer \
  --bootstrap-server localhost:9092 \
  --command-config ./config/consumer.properties  
```
