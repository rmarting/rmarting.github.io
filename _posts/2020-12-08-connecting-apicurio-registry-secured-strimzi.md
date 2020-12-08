---
layout:     post
title:      ":zap: Connecting Apicurio Registry with secured Strimzi clusters"
date:       2020-12-08 09:15
toc:        true
comments:   true
img:        apicurio-strimzi.png
tags: 
- How-to
- Apicurio
- Strimzi
- Operators
- OpenShift
- tutorial
---

[Apicurio Registry](https://www.apicur.io/registry/) is a datastore for sharing standard event schemas and API designs
across API and event-driven architectures. Apicurio Registry decouples the structure of your data from your client
applications, and enables you to share and manage your data types and API descriptions at runtime. Decoupling your
data structure from your client applications reduces costs by decreasing overall message size, creates efficiencies
by increasing consistent re-use of schemas and API designs across your organization.

Some of the most common uses cases where Apicurio Registry helps us are:

* Client applications can dynamically push or pull the latest schema updates to or from
Apicurio Registry at runtime without needing to redeploy.
* Developer teams can query the registry for existing schemas required for services already deployed in production.
* Developer teams can register new schemas required for new services in development or rolling to production.
* Store schemas used to serialize and deserialize messages, which can then be referenced from your client
applications to ensure that the messages that they send and receive are compatible with those schemas.

[Apicurio](https://www.apicur.io/) provides a open sourced Schema Registry, ready to be involved in this scenario. 

Apicurio Registry includes a set of pluggable storage options to store the APIs, rules and validations.
The [Kafka-based](https://kafka.apache.org/) storage option, provided by [Strimzi](https://strimzi.io/), is suitable for
production environments when persistent storage is configured for Kafka clusters running on [OpenShift](https://www.openshift.com/). 

In production environments security is not an option and it is something that must be provided by Strimzi for the
different components connected to. Security will be defined by:

* **Authentication** to ensure a secure client connection to the Kafka cluster.
* **Authorization** to define which users have access to which resources.

This blog post describes the high level topics to keep in mind to connect Apicurio Registry to a secure Apache Kafka cluster.

## :robot: OpenShift Operators

Strimzi and Apicurio Registry provides a set of OpenShift Operators, available through [Operator Hub](https://operatorhub.io/),
to manage the life cycle of each component. Operators are a method of packaging, deploying, and managing OpenShift applications. 

Strimzi Operators provide a set of Custom Resources Definitions (CRD) to describe the different components of the
Kafka deployment (Zookeeper, Brokers, Users, Connect, ...) These objects will provide the API to manage our Kafka cluster.

Strimzi Operators manage the authentication, authorization and users life cycle with the following custom resources:

* [Kafka Schema](https://strimzi.io/docs/operators/latest/using.html#type-Kafka-reference): Declares the Kafka topology and
features to use. This object is managed by the [Cluster Operator](https://strimzi.io/docs/operators/latest/using.html#using-the-cluster-operator-str).
* [KafkaUser Schema](https://strimzi.io/docs/operators/latest/using.html#type-KafkaUser-reference): Declares a user for
an instance of Strimzi, including the authentication, authorization and quotas definitions. This object is managed by
the [User Operator](https://strimzi.io/docs/operators/latest/using.html#assembly-using-the-user-operator-str).

[Apicurio Registry Operator](https://www.apicur.io/registry/docs/apicurio-registry/1.3.3.Final/getting-started/assembly-installing-registry-openshift.html#installing-registry-operatorhub)
provides a set of CRD to describe the different components of the Apicurio Registry deployment (storage, security,
replicas, ...) These objects will provide the API to manage our Apicurio Registry instance.

Apicurio Registry Operator manages the Apicurio Registry life cycle with the following custom resources:

* [ApicurioRegistry Schema](https://github.com/Apicurio/apicurio-registry-operator/blob/master/deploy/crds/apicur.io_apicurioregistries_crd.yaml): 
Declares the Apicurio Registry topology and main features to use. This object is managed by the Apicurio Operator.

## :cop: Authentication

Strimzi supports the following authentication mechanisms:

* SASL SCRAM-SHA-512
* TLS client authentication
* OAuth 2.0 token based authentication

These mechanisms are declared in the ```authentication``` block in the ```Kafka``` definition for each listener.
Each listener will implement the authentication mechanism defined so the client applications must authenticate with
the mechanism identified.

How to activate each mechanism in the Strimzi cluster is described below.

On the other hand we need to identify in the Apicurio Registry the authentication mechanism activated in the
Strimzi cluster. Apicurio Registry only allows the following authentication mechanisms:

* SCRAM-SHA-512
* TLS

### Strimzi Authentication

**Using SCRAM-SHA-512**

The following ```Kafka``` definition declares a Kafka cluster secured with ```SCRAM-SHA-512``` authentication
for the secured listener (TLS):

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: my-kafka
spec:
  kafka:
    listeners:
      tls:
        authentication:
          type: scram-sha-512
```

Applying this configuration will create a set of :secret: secrets where TLS certificates are stored. The secret we need to know to
allow the secured connections is declared as ```my-kafka-cluster-ca-cert```. We will need this value later.

The following ```KafkaUser``` definition declares a Kafka user with ```SCRAM-SHA-512``` authentication:

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaUser
metadata:
  name: service-registry-scram
  labels:
    strimzi.io/cluster: my-kafka
spec:
  authentication:
    type: scram-sha-512
```

Applying this configuration a new :secret: secret is created, with the same name of the user, where credentials are stored. This
secret contains the generated password to authenticate to the Kafka cluster.

```bash
❯ oc get secrets
NAME                    TYPE      DATA   AGE
service-registry-scram  Opaque    1      4s
```

**Using TLS**

The following ```Kafka``` definition declares a Kafka cluster secured with TLS authentication for the secured listener (TLS):

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: my-kafka
spec:
  kafka:
    listeners:
      tls:
        authentication:
          type: tls
```

Applying this configuration will create a set of :secret: secrets where TLS certificates are stored. The secret we need to know to
allow the secured connections is declared as ```my-kafka-cluster-ca-cert```. We will need this value later.

The following ```KafkaUser``` definition declares a Kafka user with TLS authentication:

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaUser
metadata:
  name: service-registry-tls
  labels:
    strimzi.io/cluster: my-kafka
spec:
  authentication:
    type: tls
```

Applying this configuration will create a :secret: secret, with the same name of the user, where user credentials
are stored. This secret contains the valid client certificates to authenticate to the Kafka cluster.

```bash
❯ oc get secrets
NAME                    TYPE      DATA   AGE
Service-registry-tls    Opaque    1      4s
```

### Apicurio Registry Authentication

Identifying the authentication mechanism activated in the Strimzi cluster, we need to deploy the
Apicurio Registry defining accordingly the ```ApicurioRegistry``` definition.

**NOTE**: Apicurio Registry can only connect, at the time of writing this blog, to Strimzi
*tls* listener (normally in 9093 port), whatever the authentication mechanism is activated in that listener.
It means that the ```boostrapServers``` property in ```ApicurioRegistry``` must point to that listener port:

```yaml
apiVersion: apicur.io/v1alpha1
kind: ApicurioRegistry
metadata:
  name: service-registry
spec:
  configuration:
    persistence: "streams"
    streams:
      bootstrapServers: "my-kafka-kafka-bootstrap:9093"
```

**Using SCRAM-SHA-512**

The following ```ApicurioRegistry``` definition declares a secured connection with a user with ```SCRAM-SHA-512``` authentication:

```yaml
apiVersion: apicur.io/v1alpha1
kind: ApicurioRegistry
metadata:
  name: service-registry
spec:
  configuration:
    persistence: "streams"
    streams:
      bootstrapServers: "my-kafka-kafka-bootstrap:9093"
      security:
        scram:
          user: service-registry-scram
          passwordSecretName: service-registry-scram
          truststoreSecretName: my-kafka-cluster-ca-cert
```

The values that we need to identify in this object are:

* **user**: Name of the user to connect.
* **passwordSecretName**: Name of the secret where the password is saved.
* **truststoreSecretName**: Name of secret with the CA certs of the Kafka cluster deployed.

**Using TLS**

The following ```ApicurioRegistry``` definition declares a secured connection with a user with TLS authentication:

```yaml
apiVersion: apicur.io/v1alpha1
kind: ApicurioRegistry
metadata:
  name: service-registry
spec:
  configuration:
    persistence: "streams"
    streams:
      bootstrapServers: "my-kafka-kafka-bootstrap:9093"
      security:
        tls:
          keystoreSecretName: service-registry-tls
          truststoreSecretName: my-kafka-cluster-ca-cert
```

The values that we need to identify in this object are:

* **keystoreSecretName**: Name of the user secret with the client certificates.
* **truststoreSecretName**: Name of secret with the CA certs of the Kafka cluster deployed.

## :passport_control: Authorization

Strimzi supports authorization using ```SimpleACLAuthorizer``` globally for all listeners used for
client connections. This mechanism uses Access Control Lists (ACLs) to define which users have access to which resources.

Denying is the default ACL if authorization is applied in the Kafka cluster. That requires to
declare the different rules for each user that wants to operate with the Kafka cluster.

The following ```Kafka``` definition activates authorization in the Kafka cluster:

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: my-kafka
spec:
  kafka:
    authorization:
      type: simple
```

ACLs are declared for each user in the ```KafkaUser``` definition in the ```acls``` section. That
section includes a list of resources to declare, each of them as a new rule:

* **Resource Type**: Identifies the type of object managed in Kafka, objects such as topics, consumer groups, cluster,
transaction ids and delegation tokens.
* **Name of the resource**: Identifies the resource where to apply the rule. It could be defined as literal, to identify
one resource, or a prefix pattern, to identify a list of resources.
* **Operation**: Which kind of operation is allowed to do. A full list of operations available for each resource
type is available [here](https://strimzi.io/docs/operators/latest/using.html#acl_rules).

To allow the Apicurio Registry user to work successfully with our secured Strimzi cluster we must declare
the following list of rules to specify what the user is allowed to do:

* Read its own consumer group.
* Create, read, write and describe on global ids topic (*global-id-topic*).
* Create, read, write and describe on storage topic (*storage-topic*).
* Create, read, write and describe on its own local changelog topics.
* Describe and write transactional ids on its own local group.
* Read on consumer offset topic (*__consumer_offsets*).
* Read on transaction state topics (*__transaction_state*).
* Idempotently write on cluster.

ACLs will be similar to the following definition:

```yaml
    acls:
      # Group Id to consume information for the different topics used by the Service Registry.
      # Name equals to metadata.name property in ApicurioRegistry object
      - resource:
          type: group
          name: service-registry
        operation: Read
      # Rules for the Global global-id-topic
      - resource:
          type: topic
          name: global-id-topic
        operation: Read
      - resource:
          type: topic
          name: global-id-topic
        operation: Describe
      - resource:
          type: topic
          name: global-id-topic
        operation: Write
      - resource:
          type: topic
          name: global-id-topic
        operation: Create
      # Rules for the Global storage-topic
      - resource:
          type: topic
          name: storage-topic
        operation: Read
      - resource:
          type: topic
          name: storage-topic
        operation: Describe
      - resource:
          type: topic
          name: storage-topic
        operation: Write
      - resource:
          type: topic
          name: storage-topic
        operation: Create
      # Rules for the local topics created by our Service Registry instance
      # Prefix value equals to metadata.name property in ApicurioRegistry object
      - resource:
          type: topic
          name: service-registry-
          patternType: prefix
        operation: Read
      - resource:
          type: topic
          name: service-registry-
          patternType: prefix
        operation: Describe
      - resource:
          type: topic
          name: service-registry-
          patternType: prefix
        operation: Write
      - resource:
          type: topic
          name: service-registry-
          patternType: prefix
        operation: Create
      # Rules for the local transactionalsIds created by our Service Registry instance
      # Prefix equals to metadata.name property in ApicurioRegistry object
      - resource:
          type: transactionalId
          name: service-registry-
          patternType: prefix
        operation: Describe
      - resource:
          type: transactionalId
          name: service-registry-
          patternType: prefix
        operation: Write
      # Rules for internal Apache Kafka topics
      - resource:
          type: topic
          name: __consumer_offsets
        operation: Read
      - resource:
          type: topic
          name: __transaction_state
        operation: Read
      # Rules for Cluster objects
      - resource:
          type: cluster
        operation: IdempotentWrite
```

Activating authorization in Strimzi has no effect in the ```ApicurioRegistry``` definition, because 
this is only related to the correct ACL definitions in ```KafkaUser``` objects.

## :bookmark_tabs: Summary

Apicurio Registry includes the security capabilities to connect to secure Strimzi clusters, so your production
environment could complain about your security requirements easily. This blog post demonstrates that these
technical components in your event-driven architecture are concerned with security requirements and allow you
to apply successfully.

To deeper understand and analysis, please, refer to the following references:

* [Using the User Operator](https://strimzi.io/docs/operators/latest/using.html#assembly-using-the-user-operator-str)
* [Security options for Kafka](https://strimzi.io/docs/operators/latest/using.html#assembly-securing-kafka-brokers-str)
* [Introduction to Apicurio Registry](https://www.apicur.io/registry/docs/apicurio-registry/1.3.3.Final/getting-started/assembly-intro-to-the-registry.html)

Enjoying API eventing :smiley:!!!
