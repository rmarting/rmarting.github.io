---
layout:      post
title:       ":tada: New Red Hat OpenShift Application Services Cheat Sheet!"
subtitle:    New Red Hat OpenShift Application Services Cheat Sheet available for you.
description: New Red Hat OpenShift Application Services Cheat Sheet available for you.
date:        2022-10-28 09:00:00 +0200
toc:         false
comments:    true
img:         cheat-sheet.avif
fig-caption: Blogging
fig-copy:    true
fig-author:       Pixabay
fig-author-link:  https://www.pexels.com/@pixabay
fig-gallery:      Pexels
fig-gallery-link: https://www.pexels.com/
tags:
- How-to
- Cheat Sheet
- Red Hat OpenShift
- Application Services
- tutorial
---

I was playing, testing, and learning for a while the
[Red Hat Cloud Services](https://www.redhat.com/en/technologies/cloud-computing/openshift/cloud-services).
These services include a managed platform and data services to reduce the operational cost and
complexity of delivering cloud-native applications. And also facilitates the life of the developers 😄.

Some of these services are:

* [OpenShift Streams for Apache Kafka](https://developers.redhat.com/products/red-hat-openshift-streams-for-apache-kafka/overview)
provides a managed service of Apache Kafka. You don't need to deal with the complexity of the infrastructure of an Apache Kafka cluster.
* [OpenShift Service Registry](https://developers.redhat.com/articles/2021/10/04/get-started-openshift-service-registry)
provides a full managed service of an API and schema registry, key for any event-driven architecture.

So I decided to refactor my loved 😍 [Kafka Clients Quarkus Edition](https://github.com/rmarting/kafka-clients-quarkus-sample)
repository to use these streaming services, running everything in my [Developer Sandbox](https://developers.redhat.com/developer-sandbox)
as Red Hat Developer.

The results of this learning path is this new
[Kafka Clients Quarkus Edition with Managed Services](https://github.com/rmarting/quarkus-streaming-managed-services-sample)
repository using the latest versions of the components (e.g: [Quarkus](https://quarkus.io/), [JKube](https://www.eclipse.org/jkube/))
integrated easily, running locally or remotely successfully.

Red Hat OpenShift Cloud Services provides a powerful command line interface (CLI) called `rhoas`. This
CLI is very well documented in its [website](https://appservices.tech/), however, I decided to create
my own Cheat Sheet to know all the commands and for my own references. So,

:tada: I am pleasure to announce that a new Cheat Sheet is available: :tada:

* :bookmark: [Red Hat OpenShift Application Services](/cheat-sheets/rhoas)

I hope this new cheat sheet helps you when you need to manage your own Managed Services provided by
Red Hat.

My full list of Cheat Sheets are available for your records [here](/cheat-sheets). As usual,
comments, ideas, PRs are welcomed!

Happy coding !!! 💻💾💿☕
