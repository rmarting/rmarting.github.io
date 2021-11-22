---
layout:     post
title:      "Migrating Kafka clusters with MirrorMaker2 and Strimzi"
date:       2021-11-19 00:00
toc:        false
comments:   true
img:        servers-data-center.jpg
fig-caption: Server racks on Data Center
fig-copy:    true
fig-author:       Brett Sayless
fig-author-link:  https://www.pexels.com/@brett-sayles
fig-gallery:      Pexels
fig-gallery-link: https://www.pexels.com/
tags: 
- Strimzi
- Red Hat AMQ Streams
- Migration
- Apache Kafka
- Apache MirrorMaker2
---

In this article we (my colleague [Manuel Schindler](https://www.linkedin.com/in/manuel-schindler-aa0397118/)
and I) would like to focus on the most common challenges to migrate Apache
Kafka clusters between different OpenShift platforms, and how to overcome
them by using Apache MirrorMaker and [Strimzi Operators](https://strimzi.io/).

**:newspaper_roll: :information_source: LATEST NEWS :information_source: :newspaper_roll:** This article has
been accepted and published in the [Strimzi Blog](https://strimzi.io/blog/2021/11/22/migrating-kafka-with-mirror-maker2/)
community :tada: [here](). We are glad :star_struck: to help and contribute in the Strimzi Community with our
experience and knowledge of this amazing Open Source project.

**Thank you so much Strimzi Community**!!! :muscle: :tada:

### Greetings

Many thanks to some great colleagues for help us reviewing this content:

* Hugo Guerrero [<i class="fa fa-twitter"></i>](https://twitter.com/hguerreroo) [<i class="fa fa-linkedin"></i>](https://www.linkedin.com/in/hugoguerrero/)
* Jakub Scholz [<i class="fa fa-twitter"></i>](https://twitter.com/scholzj) [<i class="fa fa-linkedin"></i>](https://www.linkedin.com/in/scholzj/)
* Rafael Yanez [<i class="fa fa-linkedin"></i>](https://www.linkedin.com/in/ryanezillescas/)
* Paolo Patierno [<i class="fa fa-twitter"></i>](https://twitter.com/ppatierno) [<i class="fa fa-linkedin"></i>](https://www.linkedin.com/in/paolopatierno/) 
