---
layout:     page
title:      "My own ActiveMQ Artemis Cheat Sheet"
permalink:  /cheat-sheets/activemq-artemis
date:       2020-09-29 20:00
published:  true
status:     publish
categories: 
- cheat-sheet
- activemq-artemis
- tutorial
tags: []
author:     rmarting
comments:   true
---

[ActiveMQ Artemis](https://activemq.apache.org/components/artemis/) is a high-perfomance messaging system
for higly scalable microservices or asynchrnous messaging between different systems.

This cheat sheet includes the most commond commands to install, deploy, administrate or
operate a messaging system based in ActiveMQ Artemis.

**NOTE**: ActiveMQ Artemis is the upstream project of
[Red Hat AMQ 7 Broker](https://access.redhat.com/products/red-hat-amq#broker-gs), so these commands are also
valid for this product.

Enjoy it !!! :cool: :sunglasses:

## Symmetric Cluster and Replicated Journal

The topology based in:

* Set of live brokers with network journal replication (Symmetric Cluster)
* Set of backup brokers as backup for the live brokers (Failover)

This diagram shows us this topology:

{:refdef: style="text-align: center;"}
[![](/images/activemq-artemis/artemis-journal-replication.png "gitmoji commit")]({{site.url}}/images/activemq-artemis/artemis-journal-replication.png)
{: refdef}

## Symmetric Cluster and Shared Journal

The topology based in:

* Set of live brokers with journal shared (Symmetric Cluster)
* Set of backup brokers as backup for the live brokers (Failover)

This diagram shows us this topology:

{:refdef: style="text-align: center;"}
[![](/images/activemq-artemis/artemis-shared-journal.png "gitmoji commit")]({{site.url}}/images/activemq-artemis/artemis-shared-journal.png)
{: refdef}

## Create a Live/Backup pair with Sharing Storage

To create the live broker:

```bash
❯ $ARTEMIS_HOME/bin/artemis create /opt/brokers/master \
--http-host $HOSTNAME \
--host $HOSTNAME \
--aio \
--clustered \
--cluster-user $ARTEMIS_CLUSTER_USER \
--cluster-password $ARTEMIS_CLUSTER_PASSWORD \
--name master \
--max-hops 1 \
--user $ARTEMIS_ADMIN_USER \
--password $ARTEMIS_ADMIN_PASSWORD \
--require-login \
--port-offset 0 \
--no-autocreate \
--data $ARTEMIS_SHARED_STORAGE_PATH \
--ssl-key $ARTEMIS_CERTS_PATH/broker.ks \
--ssl-key-password $ARTEMIS_CERTS_PASSWORD \
--shared-store \
--staticCluster $ARTEMIS_STATIC_CLUSTER_LIST
```

To create the backup broker:

```bash
❯ $ARTEMIS_HOME/bin/artemis create /opt/brokers/backup \
--http-host $HOSTNAME \
--host $HOSTNAME \
--aio \
--clustered \
--cluster-user $ARTEMIS_CLUSTER_USER \
--cluster-password $ARTEMIS_CLUSTER_PASSWORD \
--name backup \
--max-hops 1 \
--user $ARTEMIS_ADMIN_USER \
--password $ARTEMIS_ADMIN_PASSWORD \
--require-login \
--port-offset 0 \
--no-autocreate \
--data $ARTEMIS_SHARED_STORAGE_PATH \
--ssl-key $ARTEMIS_CERTS_PATH/broker.ks \
--ssl-key-password $ARTEMIS_CERTS_PASSWORD \
--shared-store \
--slave \
--staticCluster $ARTEMIS_STATIC_CLUSTER_LIST
```

Where:

* **ARTEMIS_STATIC_CLUSTER_LIST**: Identifies the static list of other live brokers of this cluster. This variable
is a value similar to: ```tcp://live-master-1:61616,...,tcp://live-master-n:61616```

## Mask passwords

To mask password to be added in ```broker.xml``` file:

```bash
❯ $ARTEMIS_HOME/bin/artemis mask $PASSWORD
``` 

References:

* [Generating a masked password](https://activemq.apache.org/components/artemis/documentation/latest/masking-passwords.html#generating-a-masked-password)

## Message Redistribution

To enable [messaging redistribution](https://activemq.apache.org/components/artemis/documentation/latest/clusters.html#message-redistribution) between brokers in a cluster, the **redistribution-delay** property
must be enabled to zero in the ```<address-setting>``` in ```broker.xml``` file:

```xml
<!--default for catch all-->
<address-setting match="#">
  <redistribution-delay>0</redistribution-delay>
</address-setting>
```

## Modularizing Broker Configuration

ActiveMQ Artemis supports XML inclusions so the configuration can be broken out into separate files. It
is an interesting feature to prevent errors and to ease populating common configuration between
clustered brokers. 

By default, the ```etc/broker.xml``` file declares ```XInclude XML``` namespace.

```xml
<configuration xmlns="urn:activemq"
           	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           	xmlns:xi="http://www.w3.org/2001/XInclude"
                ...
```

The addresses tag will be moved to an external directory in each host. The live and backup brokers
of the same host will share an ```addresses.xml``` configuration file.

```bash
$ cat /opt/amq/configs/addresses.xml
<addresses xmlns="urn:activemq:core">
   <address name="DLQ">
  	   <anycast>
     	<queue name="DLQ" />
  	   </anycast>
   </address>
   <address name="ExpiryQueue">
  	   <anycast>
     	<queue name="ExpiryQueue" />
  	   </anycast>
   </address>

   <!-- Queues and Topics -->
   <address name="SampleQueue">
  	   <anycast>
     	<queue name="SampleQueue" />
  	   </anycast>
   </address>
   <address name="SampleTopic">
  	   <multicast/>
   </address>
</addresses>
```

This file must be copied in any ActiveMQ Artemis cluster hosts. Finally each ```broker.xml``` file
will include it using the ```xi:include``` tag: 

```xml
<xi:include href=”/opt/amq/configs/addresses.xml”/>
```

**WARNING**:	External files are not monitored by the broker so it is needed to refresh or
restart it to take the latest status. ```touch``` command could be useful to modify the
timestamp of the broker.xml configuration file:

```bash
$ touch etc/broker.xml 
```

## Install as an OS service (RHEL)

Create a file called ```artemis.service``` file in the ```/etc/systemd/system``` folder. This file
will have the following content:

```text
[Unit]
Description = ActiveMQ Artemis - Broker
After = syslog.target network.target

[Service]
ExecStart = /opt/artemis/brokers/live/bin/artemis run
ExecStop = /opt/artemis/brokers/live/bin/artemis stop

User = amq-broker
Group = amq-broker

SuccessExitStatus = 0 143
RestartSec = 60
Restart = on-failure

LimitNOFILE = 102642

[Install]
WantedBy = multi-user.target
```

To Enable the OS services:

```bash
❯ systemctl enable artemis
```

To start, stop, check the status or restart the OS service:

```bash
❯ systemctl (start|stop|status|restart) artemis
```

## Producer Commands

To send messages to a resource:


```bash
❯ ./bin/artemis producer --url $ARTEMIS_CORE_PROTOCOL_URL --destination $ARTEMIS_RESOURCE \
  --user $ARTEMIS_ADMIN_USER --password $ARTEMIS_ADMIN_PASSWORD \
  --message-size 1024 --message-count 10
```

Where:

* **ARTEMIS_CORE_PROTOCOL_URL**: Identifies a connection string using the Core protocol. Valid values:
  - Single broker: ```tcp://HOSTNAME1:61616?ha=true&reconnectAttempts=3```
  - Failover connection: ```(tcp://HOSTNAME1:616161,tcp://HOSTNAME2:61616)?ha=true&reconnectAttempts=3```
* **ARTEMIS_RESOURCE**: Identifies the resource. Valid values:
  - Queue: ```queue://SampleQueue```
  - Topic: ```queue://SampleTopic```

## Consumer Commands

To consume messages from a resource:

```bash
❯ ./bin/artemis consumer --url $ARTEMIS_CORE_PROTOCOL_URL --destination $ARTEMIS_RESOURCE \
  --user $ARTEMIS_ADMIN_USER --password $ARTEMIS_ADMIN_PASSWORD \
  --message-count 10
```

Where:

* **ARTEMIS_CORE_PROTOCOL_URL**: Identifies a connection string using the Core protocol. Valid values:
  - Single broker: ```tcp://HOSTNAME1:61616?ha=true&reconnectAttempts=3```
  - Failover connection: ```(tcp://HOSTNAME1:616161,tcp://HOSTNAME2:61616)?ha=true&reconnectAttempts=3```
* **ARTEMIS_RESOURCE**: Identifies the resource. Valid values:
  - Queue: ```queue://SampleQueue```
  - Topic: ```queue://SampleTopic```

## AMQP Secured Connection

Client connection string for AMQP secure protocol:

```text
amqps://HOSTNAME:5671?sslEnabled=true&transport.trustAll=true&transport.verifyHost=false
```

## Clustered Messages Grouping

This feature allows to process messages with a particular group ID in the same order by the
consumers. Each clustered broker therefore uses a grouping handler to manage the complexity of routing
of grouped messages. Each clustered broker should chooke should choose a grouping hanlder type: Local or Remote.

Reference: [Clustered Messaging Grouping](https://activemq.apache.org/components/artemis/documentation/latest/message-grouping.html)

Local grouping handler broker:

```xml
<grouping-handler name="my-grouping-handler">
	<type>LOCAL</type>
	<address>amq-ha-cluster</address>
	<timeout>5000</timeout>
</grouping-handler>
```

Remote grouping handler broker:

```xml
<grouping-handler name="my-grouping-handler">
  <type>REMOTE</type>
  <address>amq-ha-cluster</address>
  <timeout>5000</timeout>
</grouping-handler>
```

To produce grouped messages:

```bash
❯ ./bin/artemis producer --url $ARTEMIS_CORE_PROTOCOL_URL --destination $ARTEMIS_RESOURCE \
  --user $ARTEMIS_ADMIN_USER --password $ARTEMIS_ADMIN_PASSWORD \
  --message "MSG GROUP 12" --message-count 10 --group mygroup12 -verbose
```

To consume grouped messages:

```bash
❯ /bin/artemis consumer --url $ARTEMIS_CORE_PROTOCOL_URL --destination $ARTEMIS_RESOURCE \
  --user $ARTEMIS_ADMIN_USER --password $ARTEMIS_ADMIN_PASSWORD \
  --message-count 10 --verbose
```

## Monitoring

ActiveMQ Artemis includes a [Jolokia](https://jolokia.org/) endpoints to execute administrative tasks
or query administrative information.

Samples:

* Query the up time of the broker:

```bash
❯ curl -u monitor:changeit https://HOSTNAME:8443/console/jolokia/read/org.apache.activemq.artemis:broker=%22master%22/Uptime | jq
{
  "request": {
    "mbean": "org.apache.activemq.artemis:broker=\"master\"",
    "attribute": "Uptime",
    "type": "read"
  },
  "value": "4 hours 41 minutes",
  "timestamp": 1537958954,
  "status": 200
}
```

* Query to get the total number of messages added:

```bash
❯ curl -u monitor:changeit https://HOSTNAME:8443/console/jolokia/read/org.apache.activemq.artemis:broker=%22master%22/TotalMessagesAdded | jq
{
  "request": {
    "mbean": "org.apache.activemq.artemis:broker=\"master\"",
    "attribute": "TotalMessagesAdded",
    "type": "read"
  },
  "value": 6021,
  "timestamp": 1537958962,
  "status": 200
}
```

## Performance and Limits

ActiveMQ Artemis manages several resources (descriptors, connections, …) and it is needed
to define the OS limits to allow it for the user that runs it.

Review the ```/etc/security/limits.conf``` file to add the following definition for this user:

```text
amq-broker    	soft	nofile      	65001
amq-broker    	hard	nofile      	65001
```

**WARNING**: In RHEL 8 this step is no longer needed since nofile defaults have been increased to 1048576 max open files.

ActiveMQ Artemis includes a general server thread pool used for most asynchronous actions on the
server side. This pool is defined by default to use only 30 threads and it is very useful to improve the performance.

Definition at ```broker.xml``` file:

```xml
<thread-pool-max-size>120</thread-pool-max-size>
```

There are a few things that can go wrong in a production environment (bugs, IO errors, memory issues, …),
so ActiveMQ Artemis includes a protection to shut itself down when bad things happen
(as a safeguard). This method includes different policies:

* **LOG** (default): Log messages into ```artemis.log``` to inform that something is wrong.
* **HALT** (default at broker creation): Stop the messaging process but not the VM.
* **SHUTDOWN**: Shutdown the VM process.

To check easily if a broker suffered an issue the best practice is to use SHUTDOWN policy. It is
very easy to check if the broker is running or not checking the service or the java process in OS.

Definition at ```broker.xml``` file:

```xml
<critical-analyzer-policy>SHUTDOWN</critical-analyzer-policy>
```

ActiveMQ Artemis is defined to automatically create an address/queue when a new sender/receiver is
connected. It is a great feature because it allows us to avoid having to manage the address in the
```broker.xml``` file. However ActiveMQ Artemis also deletes an address/queue when there is
not a sender/receiver connected and there are no messages persisted. This feature is also
great however it includes some extra staff to manage this process.

Definition at ```broker.xml``` file:

```xml
<address-setting match="#">
  <!-- Others properties -->
  <auto-delete-queues>false</auto-delete-queues>
  <auto-delete-addresses>false</auto-delete-addresses>
  <!-- Others properties -->
</address-setting>
```

Main References:

* [Performance Tuning](https://activemq.apache.org/components/artemis/documentation/latest/perf-tuning.html)
* [Thread management](https://activemq.apache.org/components/artemis/documentation/latest/thread-pooling.html)
* [Critical Analysis of the broker](https://activemq.apache.org/components/artemis/documentation/latest/critical-analysis.html)

## Managing users

Add a user:

```bash
❯ /opt/artemis/brokers/live/bin/artemis user add --user user1 --password user1-password1 --role role1
```

Reset a user (change user password and/or role/s):

```bash
❯ /opt/artemis/brokers/live/bin/artemis user reset --user user1 --password user1-password2 --role role2,role3
```

Remove a user:

```bash
❯ /opt/artemis/brokers/live/bin/artemis user rm --user user1
```
