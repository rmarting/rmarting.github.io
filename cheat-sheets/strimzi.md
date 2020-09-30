---
layout:     post
title:      "My own Strimzi Cheat Sheet"
toc:        true
comments:   true
permalink:  /cheat-sheets/strimzi
tags: 
- How-to
- Cheat Sheet
- Strimzi
- tutorial
---

[Strimzi](https://kafka.apache.org/) is an open-source project that provides a way to run an
Apache Kafka cluster on Kubernetes in various deployment configurations.

This cheat sheet includes the most commond commands to install, deploy, administrate or
operate an Apache Kafka cluster using Strimzi operators.

**NOTE**: Apache Kafka is the upstream project of
[Red Hat AMQ Streams](https://access.redhat.com/products/red-hat-amq#streams), so these commands are also
valid for this product.

To execute most of the commands listed here require a Kubernetes (kubectl) or OpenShift CLI (oc). 

Enjoy it !!! :cool: :sunglasses:

## Linux Container Images

Strimzi Images are published in [Docker Hub](https://hub.docker.com/r/strimzi/kafka/tags). The main images
availables are:

* ```podman pull docker.io/strimzi/kafka:latest-kafka-2.6.0```
* ```podman pull docker.io/strimzi/kafka:latest-kafka-2.5.0```
* ```podman pull docker.io/strimzi/kafka:latest-kafka-2.4.0```

Red Hat AMQ Streams images are published in [Red Hat Container Catalog](https://catalog.redhat.com/software/containers/explore).
The main versions availables are:

* ```podman pull registry.redhat.io/amq7/amq-streams-kafka-25-rhel7```
* ```podman pull registry.redhat.io/amq7/amq-streams-kafka-24-rhel7```

To pull images from Red Hat Container Catalog you must need a valid user authenticated on it.

## Sample Producers

To execute a single kafka console producer:

```bash
❯ oc run kafka-console-producer -ti --image=$STRIMZI_IMAGE \
  --rm=true --restart=Never \
  -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic sample-topic
```

To execute a producer performance test:

```bash
❯ oc run kafka-producer-perf-test -ti --image=$STRIMZI_IMAGE \
  --rm=true --restart=Never \
  -- bin/kafka-producer-perf-test.sh --topic my-topic-sample --num-records 100000 --throughput 1000 --record-size 2048 --print-metrics --producer-props bootstrap.servers=my-cluster-kafka-bootstrap:9092
```

If Apache Kafka requires SASL authentication with SCRAM-SHA-512 the following properties should be passed:

```text
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=user password=StrimziIsTheBest;"
```

## Sample Consumers

To execute a single kafka console consumer:

```bash
❯ oc run kafka-console-consumer -ti --image=$STRIMZI_IMAGE \
  --rm=true --restart=Never \
  -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic-sample --from-beginning
```

To execute a consumer performance test:

```bash
❯ oc run kafka-consumer-perf-test -ti --image=$STRIMZI_IMAGE \
  --rm=true --restart=Never \
  -- bin/kafka-consumer-perf-test.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic my-topic-sample --from-latest --messages 100000 --print-metrics --show-detailed-stats
```

If Apache Kafka requires SASL authentication with SCRAM-SHA-512 the following properties should be passed:

```text
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=user password=StrimziIsTheBest;"
```

## Rolling Updates

To update Kafka StatefulSet without downtime use the following command:

```bash
❯ oc annotate statefulset my-cluster-kafka strimzi.io/manual-rolling-update=true
```

To update Zookeeper StatefulSet without downtime use the following command:

```bash
❯ oc annotate statefulset my-cluster-zookeeper strimzi.io/manual-rolling-update=true
```

**Note**: The process can take up to 2 minutes to start, as the operator detects the
annotation. The annotation is removed after the rolling update process is complete.

Reference:

* [Strimzi / Performing a rolling update of a Kafka cluster](https://strimzi.io/docs/operators/master/using.html#proc-manual-rolling-update-kafka-deployment-configuration-kafka)

## Topic Managment

* Create a topic using a KafkaTopic resource:

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaTopic
metadata:
  labels:
    strimzi.io/cluster: my-cluster
  name: sample-topic
spec:
  config:
    cleanup.policy: delete
    delete.retention.ms: '7200000'
    max.message.bytes: '52428800'
    message.format.version: 2.4-IV1
    min.insync.replicas: '2'
  partitions: 1
  replicas: 3
  topicName: sample-topic
```

## User Management

* Create a consumer user with authentication:

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaUser
metadata:
  name: user
  labels:
    strimzi.io/cluster: my-cluster
spec:
  authentication:
    type: scram-sha-512
  authorization:
    type: simple
```

ACLs to read from a topic from an specific consumer group:

```yaml
    acls:
      # Consumer ACLs
      - resource:
          type: topic
          name: sample-topic
          patternType: literal
        operation: Read
        host: "*"
      - resource:
          type: topic
          name: sample-topic
          patternType: literal
        operation: Describe
        host: "*"
      - resource:
          type: group
          name: consumer-group
          patternType: literal
        operation: Read
		host: "*"
```

ACLs to write messages into a topic:

```yaml
    acls:
      # Producer ACLs
      - resource:
          type: topic
          name: sample-topic
          patternType: literal
        operation: Create
        host: "*"
      - resource:
          type: topic
          name: sample-topic
          patternType: literal
        operation: Write
        host: "*"
      - resource:
          type: topic
          name: sample-topic
          patternType: literal
        operation: Describe
        host: "*"
```

## Change User Password

Encode the new password:

```bash
❯ echo -n 'StrimziIsTheBest' | base64
U3RyaW16aUlzVGhlQmVzdA==
```

**Note**: Passwords must consist of letters (upper or lower case) and have an exact length of 12 characters. Otherwise it will be replaced by the Strimzi operator.

**Note**: The “-n” option is important as it eliminates the line break.

Edit the secret with the same name as the KafkaUser:

```yaml
apiVersion: v1
data:
  password: U3RyaW16aUlzVGhlQmVzdA==
kind: Secret
metadata:
  name: user
```

Perform a rolling update of the Zookeeper StatefulSet:

```bash
❯ oc annotate statefulset my-cluster-zookeeper strimzi.io/manual-rolling-update=true
```

## Recreate Apache Kafka cluster

Deleting Kafka CR will delete everything created previously (e.g: certificates). If you want
to recreate an Apache Kafka cluster reusing the previous certificates (used and shared in other systems),
follow the next steps:

* Disable automatic certificate creation in Kafka definition and define a validation for 10 years, or the
time you need:

```yaml
kind: Kafka
version: kafka.strimzi.io/v1beta1
spec:
  # ...
  clusterCa:
    generateCertificateAuthority: false
    validityDays: 3650
  clientsCa:
    generateCertificateAuthority: false
    validityDays: 3650
```

* Create a backup of the current certs:

```bash
❯ oc get --export -o yaml $(oc get secrets -o name | grep -E "my-cluster-cluster|my-cluster-clients") > certs.yaml
```

* Create a backup of the current cluster:

Now to recreate the cluster we only need to backup the Kafka monitor resource, delete it and create it again.

```bash
❯ oc get --export -o yaml kafka my-cluster > my-cluster-backup.yml
```

* Delete the cluster:

```bash
❯ oc delete kafka my-cluster
```

* Create the cluster:

```bash
❯ oc create -f my-cluster.yml
```

References:

* [Renewing CA certificates manually](https://strimzi.io/docs/operators/master/using.html#proc-renewing-ca-certs-manually-deployment-configuration-kafka)
* [Installing your own CA certificates](https://strimzi.io/docs/operators/master/using.html#installing-your-own-ca-certificates-str)

## Backup Users and Passwords

To keep users and passwords we need both password secrets and KafkaUser resources, otherwise
Strimzi operator will create a new random password for the existing KafkaUser resources.

Backup the passwords:

```bash
❯ oc get secrets --export -o yaml --selector=strimzi.io/kind=KafkaUser > passwords-backup.yml
```

Backup the users:

```bash
❯ oc get kafkauser --export -o yaml > users-backup.yml
```

## Restore Users and Passwords

Passwords need to be created first for the exact same reason, otherwise the Strimzi operator
will create a new random password for the existing KafkaUser resources.

Create passwords: 

```bash
❯ oc create -f passwords.yml
```

Create users: 

```bash
❯ oc create -f users.yml
```

Perform a rolling update of the Zookeeper StatefulSet to update security credentials:

```bash
❯ oc annotate statefulset my-cluster-zookeeper strimzi.io/manual-rolling-update=true
```
