---
layout:     post
title:      "Blue / Green deployments :rocket: in OpenShift with Eclipse JKube"
date:       2020-10-23 10:00:00 +0200
toc:        true
comments:   true
img:        eclipse-jkube.png
tags: 
- How-to 
- OpenShift
- kubernetes
- jkube
- development
---

[Blue Green Deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html) is an application release
model very well-known that gradually transfers user traffic from a previous version of an application or microservice
to a nearly identical new release—both of which are running in production. 

The old version of the application or microservice is identified with a color (e.g: <span style="color:blue">Blue</span>)
while the new version is identified with the other color (e.g: <span style="color:green">Green</span>). This model
allows to manage the production traffic from one color (old version) to the other (new version), and the old
version can standby in case of rollback or pulled from production and updated to become the template upon
which the next update is made :dizzy:.

This model requires a Continuous Deployment model or pipeline :incoming_envelope: to orchestrate the promotion
between each color and manage the down times or rolling process. CD pipelines could be implemented using
the capabilities provided by Eclipse JKube :sparkler::bulb:.

[Eclipse JKube](https://www.eclipse.org/jkube/) is a collection of plugins that are used for building
container images using Docker, JIB or S2I build strategies. In addition, Eclipse JKube generates and
deploys Kubernetes/OpenShift manifests (YAML configuration files) at compile time too.

This article describes an approach to integrate a <span style="color:blue">Blue</span>/<span style="color:green">Green</span>
deployment with Eclipse JKube. This approach is based in
[Maven Profiles](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)
and filtering resources. Of course, this is only an integration sample and it is open to
have other approaches. For example, I would like to dig into the 
[Eclipse JKube Profiles](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin#profiles) as other
approach :+1:. However, feel free to add your ideas, comments or suggestions :cupid:.

## Identifying Blue / Green Version

Every time we want to deploy a new version it is needed to identify the right color to be
used to deploy it. This process should be done in the CD pipeline before to deploy our
next application version.

For example we could use a variable called ```app.bg.version``` in the ```pom.xml``` file. To
declare each version a Maven Profile could help us. This variable will be used to filter some
resources at deployment time.

Sample Maven profile for <span style="color:blue">Blue</span> Deployment:

{% highlight xml %}
<profile>
    <id>blue</id>
    <properties>
        <app.bg.version>blue</app.bg.version>
    </properties>
</profile>
{% endhighlight xml %}

Sample Maven profile for <span style="color:green">Green</span> Deployment:

{% highlight xml %}
<profile>
    <id>green</id>
    <properties>
        <app.bg.version>green</app.bg.version>
    </properties>
</profile>
{% endhighlight xml %}

This value could also be used to define the version of the image built. A sample of that could
be using the ```jkube.generator.name``` property in the ```pom.xml``` file as:

{% highlight xml %}
<jkube.generator.name>${project.artifactId}-${app.bg.version}:${project.version}</jkube.generator.name>
{% endhighlight %}

## Defining Blue/Green Objects

Each <span style="color:blue">Blue</span> and <span style="color:green">Green</span> version should use its own
resources at deployment time to avoid replace or redeploy the other deployment. To do that we would use
[Eclipse JKube resource fragments](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin#_resource_fragments)
to declare specific definitions of some Kubernetes or OpenShift objects.

In our case we will declare a custom deployment object creating a ```deployment.yml``` file in
the ```jkube``` folder. This custom deployment will override some properties to use the
active version at deployment time.

A sample of this file could be similar to:

{% highlight yaml %}
metadata:
  name: ${project.artifactId}-${app.bg.version}
  labels:
    group: ${project.groupId}
    project: ${project.artifactId}
    version: ${project.version}-${app.bg.version}
    provider: jkube
spec:
  template:
    spec:
      containers:
        - env:
          - name: APP_BG_VERSION
            value: ${app.bg.version}
{% endhighlight %}

To access the version deployed a service is needed to map it. This service should be
aligned with the right version. A ```service.yml``` file in the ```jkube``` folder similar
to the next one could be similar to:

{% highlight yaml %}
metadata:
  name: ${project.artifactId}-${app.bg.version}
  labels:
    group: ${project.groupId}
    project: ${project.artifactId}
    version: ${project.version}-${app.bg.version}
    provider: jkube
    expose: "true"
spec:
  selector:
    deploymentconfig: ${project.artifactId}-${app.bg.version}
{% endhighlight %}

## Deploying the Blue Version

Now, we could deploy the <span style="color:blue">Blue</span> version easily using the Maven Profile.
For example, using the [OpenShift Maven Plugin](https://www.eclipse.org/jkube/docs/openshift-maven-plugin) the
command will be similar to:

{% highlight shell %}
❯ mvn clean package oc:resource oc:build oc:apply -Pblue
{% endhighlight %}

## Deploying the Green Version

On the other hand, we could deploy the <span style="color:green">Green</span> version easily using the Maven Profile. For
example, using the [OpenShift Maven Plugin](https://www.eclipse.org/jkube/docs/openshift-maven-plugin)
the command will be similar to:

{% highlight shell %}
❯ mvn clean package oc:resource oc:build oc:apply -Pgreen
{% endhighlight %}

## Blue/Green Resources Deployed

This process generates the following objects:

* Image Streams :camera: for each version:

{% highlight shell %}
❯ oc get is
NAME                                 IMAGE REPOSITORY                                                                                              TAGS                            UPDATED
kafka-clients-quarkus-sample-blue    default-route-openshift-image-registry.apps-crc.testing/amq-streams-demo/kafka-clients-quarkus-sample-blue    1.0.0-SNAPSHOT,1.2.0-SNAPSHOT   38 hours ago
kafka-clients-quarkus-sample-green   default-route-openshift-image-registry.apps-crc.testing/amq-streams-demo/kafka-clients-quarkus-sample-green   1.1.0-SNAPSHOT,1.3.0-SNAPSHOT   38 hours ago
{% endhighlight %}

* Build Config :construction_worker: for each version:

{% highlight shell %}
❯ oc get bc
NAME                                     TYPE     FROM     LATEST
kafka-clients-quarkus-sample-blue-s2i    Source   Binary   4
kafka-clients-quarkus-sample-green-s2i   Source   Binary   2
{% endhighlight %}

* Deployments :sparkles: for each version:

{% highlight shell %}
❯ oc get dc
NAME                                 REVISION   DESIRED   CURRENT   TRIGGERED BY
kafka-clients-quarkus-sample-blue    7          1         1         config,image(kafka-clients-quarkus-sample-blue:1.0.0-SNAPSHOT)
kafka-clients-quarkus-sample-green   5          1         1         config,image(kafka-clients-quarkus-sample-green:1.1.0-SNAPSHOT)
{% endhighlight %}

* Services :eyes: for each version:

{% highlight shell %}
❯ oc get svc
NAME                                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kafka-clients-quarkus-sample-blue      ClusterIP   172.25.180.88    <none>        8181/TCP                     38h
kafka-clients-quarkus-sample-green     ClusterIP   172.25.87.170    <none>        8181/TCP                     38h
{% endhighlight %}

Thanks of all these objects we could manage both versions at the same time, and in combination with a CD pipeline and other resources
could manage the traffic to activate the right version (<span style="color:blue">Blue</span> or <span style="color:green">Green</span>)
to the final users.

For example, we could have an OpenShift route to balance the application between each versions. That route could be similar to:

{% highlight yaml %}
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: kafka-clients-quarkus-sample-lb
  labels:
    app: kafka-clients-quarkus-sample
spec:
  to:
    kind: Service
    name: kafka-clients-quarkus-sample-blue
    weight: 100
{% endhighlight %}

## Rolling from Blue to Green

To roll from <span style="color:blue">**Blue**</span> to <span style="color:green">**Green**</span> version it is
only to path the load balancer route (in the case of OpenShift) as:

{% highlight shell %}
❯ oc patch route kafka-clients-quarkus-sample-lb --type=merge -p '{"spec": {"to": {"name": "kafka-clients-quarkus-sample-green"}}}'
{% endhighlight %}

## Rolling from Green to Blue

To roll from <span style="color:green">**Green**</span> to <span style="color:blue">**Blue**</span> version it is
only to path the load balancer route (in the case of OpenShift) as:

{% highlight shell %}
❯ oc patch route kafka-clients-quarkus-sample-lb --type=merge -p '{"spec": {"to": {"name": "kafka-clients-quarkus-sample-blue"}}}'
{% endhighlight %}

## Show me the code

If you want to test and verify this approach, I developed a sample case in one of my favorite
[GitHub repo](https://github.com/rmarting/kafka-clients-quarkus-sample/tree/feature/b-g-deployment-strategy).
This repo includes amazing frameworks as [Quarkus](https://quarkus.io/), [Schemas Avro](https://avro.apache.org/)
and [Apache Kafka](https://kafka.apache.org/) in a
small [Event-Driven Architecture](https://en.wikipedia.org/wiki/Event-driven_architecture).

Enjoy it! :muscle:

## References

* [Eclipse JKube](https://www.eclipse.org/jkube/)
* [Eclipse JKube GitHub](https://github.com/eclipse/jkube)
* [What is Blue Green deployment?](https://www.redhat.com/en/topics/devops/what-is-blue-green-deployment)
* [Wikipedia - Blue-Green deployment](https://en.wikipedia.org/wiki/Blue-green_deployment)
