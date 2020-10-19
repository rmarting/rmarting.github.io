---
layout:     post
title:      ":sparkles: How to upgrade Strimzi Operator using the CLI"
date:       2020-09-25 15:00
toc:        true
comments:   true
img:        how-to-vr.jpg
tags:
- How-to
- kubernetes
- OpenShift
- operators
- Strimzi
- Red Hat AMQ Streams
---

[Operator Lifecycle Manager (OLM)]((https://docs.openshift.com/container-platform/4.5/operators/understanding_olm/olm-understanding-olm.html))
helps users install, update, and manage the lifecycle of all Operators and their associated
services running across their clusters. It is part of the [Operator Framework](https://github.com/operator-framework),
an open source toolkit designed to manage Kubernetes native applications (Operators) in
an effective, automated, and scalable way.

OLM is responsible for managing the custom resources definitions (CRDs) that are the
basis for the OLM framework:

* **OperatorGroup (og)**:	OLM	used to group multiple namespaces and prepare for use by an operator.
* **Subscription (sub)**: Catalog	used to keep CSVs up to date by tracking a channel in a package.
* **ClusterServiceVersion (csv)**:OLM	application metadata: name, version, icon, required resources, installation, etc...
* **InstallPlan (ip)**: Catalog	calculated list of resources to be created in order to automatically install/upgrade a CSV.

This article describes the upgrade process for the Strimzi Operator (or Red Hat AMQ Streams Operator) in
an OpenShift Platform.

> **Note 1**: Only *cluster-admin* users could manage the OLM objects.

> **Note 2**: The article only covers the upgrade process of the Strimzi Operator, does not cover
the strategies to upgrade the Apache Kafka cluster version.

## Operator Subscription

To deploy the Strimzi Operators only to inspect a namespace (*namespaced scope*), we need to use
an [OperatorGroup]((https://docs.openshift.com/container-platform/4.5/operators/understanding_olm/olm-understanding-operatorgroups.html)).
An ```OperatorGroup``` is an OLM resource that provides multitenant configuration to OLM-installed Operators.

This is a definition for manage operators in ```myproject``` namespace:

```yaml
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: myproject-og
  namespace: myproject
spec:
  targetNamespaces:
    - myproject
```

To install it:

```bash
❯ oc apply -f operatorgroup.yml
```

A ```Subscription``` represents an intention to install an Operator. It is the custom resource
that relates an Operator to a CatalogSource. Subscriptions describe which channel
of an Operator package to subscribe to, and whether to perform updates automatically or manually.
If set to automatic, the Subscription ensures OLM manages and upgrades the Operator to
ensure that the latest version is always running in the cluster.

This is a ```Subscription``` definition to install Strimzi Operator:

```yaml
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: strimzi-kafka-operator
  namespace: myproject
spec:
  channel: stable
  installPlanApproval: Manual
  name: strimzi-kafka-operator
  source: community-operators
  sourceNamespace: openshift-marketplace
  startingCSV: strimzi-cluster-operator.v0.17.0
```

In the case of Red Hat AMQ Streams the definition is:

```yaml
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: amq-streams
  namespace: myproject
spec:
  channel: stable
  installPlanApproval: Manual
  name: amq-streams
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  startingCSV: amqstreams.v1.4.0
```

In both cases the following attributes are the important:

The channel, such as alpha, beta, or stable, helps determine which Operator stream should be installed from the CatalogSource.

* **channel**: Identifies which Operator stream should be installed from the
CatalogSourceThe channel. Values: ```alpha```, ```beta```, or ```stable```.
* **installPlanApproval**: Identifies the approval mode of the ```installPlan```. If you defined
as ```Automatic``` then the upgrade of the operator will be done as soon a new version is available.
* **startingCSV**: Identifies the initial version to install of the operator.

:bulb: If you don't know to values needed for these attributes, the following command will show you
that information easily:

```bash
❯ oc get packagemanifest strimzi-kafka-operator \
	-o jsonpath='{range .status.channels[*]}{"Chanel: "}{.name}{" -- CSV: "}{.currentCSV}{" -- Version: "}{.currentCSVDesc.version}{" -- skipRange: "}{.currentCSVDesc.annotations.olm\.skipRange}{"\n"}'
Chanel: stable -- CSV: strimzi-cluster-operator.v0.19.0 -- Version: 0.19.0 -- skipRange: 
Chanel: strimzi-0.19.x -- CSV: strimzi-cluster-operator.v0.19.0 -- Version: 0.19.0 -- skipRange: 
```

For Red Hat AMQ Streams should be similar to:

```bash
❯ oc get packagemanifest amq-streams \
	-o jsonpath='{range .status.channels[*]}{"Chanel: "}{.name}{" -- CSV: "}{.currentCSV}{" -- Version: "}{.currentCSVDesc.version}{" -- skipRange: "}{.currentCSVDesc.annotations.olm\.skipRange}{"\n"}'
Chanel: amq-streams-1.5.x -- CSV: amqstreams.v1.5.3 -- Version: 1.5.3 -- skipRange: 
Chanel: amq-streams-1.x -- CSV: amqstreams.v1.5.3 -- Version: 1.5.3 -- skipRange: 
Chanel: stable -- CSV: amqstreams.v1.5.3 -- Version: 1.5.3 -- skipRange: 
```

:bulb: This article is focused to show how to update the Strimzi Operator manually, so this is the reason
that we will use ```Manual``` installation plan approval mode. It could be a good practice to declare
your subscriptions manually to control in which moment update your operators.

This command will install the subscription to deploy the operator:

```bash
❯ oc apply -f subscription.yml
```

After you create the subscription we could check it:

```bash
❯ oc get sub
NAME                     PACKAGE                  SOURCE                CHANNEL
strimzi-kafka-operator   strimzi-kafka-operator   community-operators   stable
```

The **install plan** is created, but it is not approved, so the operator is not installed jet:

```bash
❯ oc get ip
NAME            CSV                                APPROVAL   APPROVED
install-kt477   strimzi-cluster-operator.v0.17.0   Manual     false
```

This command will approve it to start the deployment of the operator:

```bash
❯ oc patch installplan install-kt477 --type merge --patch '{"spec":{"approved":true}}'
```

The install plan will be updated as:

```bash
❯ oc get ip
NAME            CSV                                APPROVAL   APPROVED
install-kt477   strimzi-cluster-operator.v0.17.0   Manual     true
```

After some minutes we could check the status of the CSV:

```bash
❯ oc get csv
NAME                               DISPLAY   VERSION   REPLACES   PHASE
strimzi-cluster-operator.v0.17.0   Strimzi   0.17.0               Succeeded
```

Now you are ready to deploy an Apache Kafka cluster using the Kafka CRD provided by this operator:

```bash
❯ oc apply -f kafka.yml
```

## Upgrading the Operator

When a new version of your operator is released, OLM will check it and then a new install plan
will be created for the new version.

If there is a new version available, the install plan should be similar to:

```bash
❯ oc get ip
NAME            CSV                                APPROVAL   APPROVED
install-2lxhr   strimzi-cluster-operator.v0.18.0   Manual     false
install-kt477   strimzi-cluster-operator.v0.17.0   Manual     true
```

Here you can see that there is a new ```strimzi-cluster-operator.v0.18.0``` version available, so
we could approved in the same way as the first installation:

```bash
❯ oc patch installplan install-2lxhr --type merge --patch '{"spec":{"approved":true}}'
```

When the new version is active the CSV will be updated and then you could check the upgrading history:

```bash
❯ oc get csv
NAME                               DISPLAY   VERSION   REPLACES                           PHASE
strimzi-cluster-operator.v0.17.0   Strimzi   0.17.0                                       Replacing
strimzi-cluster-operator.v0.18.0   Strimzi   0.18.0    strimzi-cluster-operator.v0.17.0   Installing
```

When the upgrade concluded the final status of the CSV will be:

```bash
❯ oc get csv
NAME                               DISPLAY   VERSION   REPLACES                           PHASE
strimzi-cluster-operator.v0.18.0   Strimzi   0.18.0    strimzi-cluster-operator.v0.17.0   Succeeded
```

The operator pod it will be recreated with the new version as you can see in the following watcher command:

```bash
❯ oc get pod -w
NAME                                                  READY   STATUS              RESTARTS   AGE
strimzi-cluster-operator-v0.17.0-5d9c7757c9-2blvf   1/1     Running             0          2m40s
strimzi-cluster-operator-v0.18.0-647d7bccc-bh66x    0/1     Pending             0          0s
strimzi-cluster-operator-v0.18.0-647d7bccc-bh66x    0/1     ContainerCreating   0          0s
strimzi-cluster-operator-v0.18.0-647d7bccc-bh66x    1/1     Running             0          2m36s
strimzi-cluster-operator-v0.17.0-5d9c7757c9-2blvf   1/1     Terminating         0          6m22s
```

This process will be repeated every time a new version of the operator is available. You only need to
approve each version.

## Versions Upgrading Workflow

Strimzi Operator is a very matured operator and includes a large list of versions. Each version
includes a set of new features. If you start from an older version, OLM will prompt you to upgrade
to the next version.

The upgrade pathway for Strimzi Operator versions starting from ```v0.17.0```:

```v0.17.0``` -> ```v0.18.0``` -> ```v0.19.0```

In the case of Red Hat AMQ Streams starting from ```v1.4.0``` version is:

```v1.4.0``` -> ```v1.4.1``` -> ```v1.5.0``` -> ```v1.5.1``` -> ```v1.5.2``` -> ```v1.5.3```

## References

* [OpenShift Documentation - Understanding OLM](https://docs.openshift.com/container-platform/4.5/operators/understanding_olm/olm-understanding-olm.html)
* [OpenShift Documentation - Operator installation and upgrade workflow in OLM](https://docs.openshift.com/container-platform/4.5/operators/understanding_olm/olm-workflow.html)
* [OpenShift Documentation - OperatorGroups](https://docs.openshift.com/container-platform/4.5/operators/understanding_olm/olm-understanding-operatorgroups.html)
* [OpenShift Documentation - OLM Architecture](https://docs.openshift.com/container-platform/4.5/operators/understanding_olm/olm-arch.html)
