---
layout:      post
title:       ":rocket: Cloud Native CICD Pipelines in OpenShift"
subtitle:    Overview of cloud native CICD pipelines provided by Tekton running on top of OpenShift.
description: Overview of cloud native CICD pipelines provided by Tekton running on top of OpenShift.
date:        2022-04-01 00:00
toc:         true
comments:    true
img:         cloud-containers-by-tekton.avif
fig-caption:      Cloud containers by Tekton
fig-copy:         true
fig-author:       Oliver Halls
fig-author-link:  https://www.pexels.com/@oliver-halls-1858500
fig-gallery:      Pexels
fig-gallery-link: https://www.pexels.com/
tags: 
- Tutorial
- OpenShift
- Operators
- Tekton
- CICD
- Cloud Native
- development
- productivity
---

## Cloud Native CICD Pipelines in OpenShift

My first [Continuous Integration](https://openpracticelibrary.com/practice/continuous-integration/) and
[Continuous Delivery](https://openpracticelibrary.com/practice/continuous-delivery/) pipelines (from now CICD)
were created with [Hudson](https://en.wikipedia.org/wiki/Hudson_(software))
(I know, I know !! I am very old :older_man: on this space), and after that with [Jenkins](https://www.jenkins.io/)
for longer time. During this long period I used it (and others similar) to build, test, package,
and deploy many different kind of applications (Monolith, SOA Services, Microservices, standalone apps, ...) into
many different kind of platforms ([Tomcat](https://tomcat.apache.org/), 
[Red Hat JBoss Enterprise Applications](https://www.redhat.com/en/technologies/jboss-middleware/application-platform),
[WebLogic](https://www.oracle.com/es/java/weblogic/), ...)
and of course on containers platform such as [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift).

However, in cloud environments with cloud native applications in some cases I found so much complexity not easy to deal
with it. Basically these tools were defined to run on Virtual Machines, required IT operations for maintenance,
conflicts between teams or projects with shared plugins or extensions, no native interoperability with Kubernetes resources, ...

... and nowadays I found a new player in this scenario to improve my CICD pipelines in the new Cloud Native World,
with containers, Kubernetes, and OpenShift. This player is [Tekton](https://tekton.dev/), or
[Red Hat OpenShift Pipelines](https://cloud.redhat.com/learn/topics/ci-cd) as the enterprise version for OpenShift.

Tekton is a cloud-native solution for building CICD systems, providing a set of building blocks, components and an
extended catalog ([Tekton Hub](https://hub.tekton.dev/)) with great resources to use, making it a complete
ecosystem. It is part of the [CD Foundation](https://cd.foundation/) with a great community, very active.

As Tekton is installed as a [Kubernetes Operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/),
providing [Custom Resources Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
to define the building blocks, it is very easy to create, and reuse them in the pipelines. As other Kubernetes or
OpenShift objects, Tekton CRDs are first-citizens, so many of the processes uses to manage your OpenShift platform
are valid for them. For example, as a fan of [Everything as Code](https://openpracticelibrary.com/practice/everything-as-code/)
practice, I can define my CICD pipelines as code and store them in a Git repository.

Tekton uses the services provides by the OpenShift, so it is designed for containers, and scalability. It means that
the pipelines and tasks are executed on-demand with containers, so it is easy to scale them. We, as CICD designers, 
don't need to deal with the platform, or infrastructure, as OpenShift provides us the services, and Tekton the objects
to design the flow of our CICD pipeline.

In that integration with OpenShift services, the building images processes are now really native and we could use
any of the technologies available, such as [source-to-image](https://github.com/openshift/source-to-image),
[buildah](https://buildah.io/), [kaniko](https://github.com/GoogleContainerTools/kaniko),
[jib](https://github.com/GoogleContainerTools/jib), ... Not more needed creating a custom image for a Jenkins-agent
to build our application.

The same to integrate the deployment processes of your application, as you can interact natively with the platform
... but in this scenario I am more fan to move the Continuous Delivery following
the [GitOps](https://openpracticelibrary.com/practice/gitops/) approach with another amazing tool
as [ArgoCD](https://github.com/argoproj/argo-cd) (but that is another story, and other blog-post :wink:).

At last, but not least, Tekton provides a set of amazing tooling to use on your favorite IDE, command-line tool
and so on, and accelerate the adoption by your side, and make your life easier:

* [`tkn` command line interface](https://tekton.dev/docs/cli/)
* [Tekton Pipelines Extension for VSCode](https://github.com/redhat-developer/vscode-tekton)
* [Tekton Pipelines by Red Hat for IntelliJ](https://plugins.jetbrains.com/plugin/14096-tekton-pipelines-by-red-hat)

So, let's go through across the main components of this amazing project.

## Tekton Components

Tekton provides a set of different components to design and build your pipelines:

* Tasks
* Pipelines
* Workspaces
* Triggers

There are others too, but these ones are the base.

### Tasks

[`Tasks`](https://tekton.dev/docs/pipelines/tasks/) is a collection of `Steps` that
you define and arrange in a specific order of execution as part of your continuous
integration flow.

`Tasks` can have more than one `step`, allowing to specialize the task with more
detailed steps. The steps will run in the order in which they are defined in the
steps array.

A `Task` is available within a specific namespace, while a `ClusterTask` is
available across the entire cluster.

A `Task` is executed as a Pod on your OpenShift cluster.

This is the typical `Hello World` Task.

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello-task
spec:
  steps:
    - name: say-hello
      image: registry.redhat.io/ubi7/ubi-minimal
      command: ['/bin/bash']
      args: ['-c', 'echo Hello World']
```

Meanwhile a `Task` is a definition, the execution of the task with the results
and outputs is a `TaskRun`.

An execution of previous task should be similar to (*simplified and omitted some fields*):

```yaml
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  generateName: hello-task-run-
  name: hello-task-run-9d8hs
  uid: f3c8d81b-3e8d-4ad5-a01b-bf9b147485f6
  creationTimestamp: '2022-03-31T16:34:41Z'
  namespace: pipelines-demo
  labels:
    app.kubernetes.io/managed-by: tekton-pipelines
    tekton.dev/task: hello-task
spec:
  taskRef:
    kind: Task
    name: hello-task
status:
  completionTime: '2022-03-31T16:34:47Z'
  conditions:
    - lastTransitionTime: '2022-03-31T16:34:47Z'
      message: All Steps have completed executing
      reason: Succeeded
      status: 'True'
      type: Succeeded
  podName: hello-task-run-9d8hs-pod-kqcjv
  startTime: '2022-03-31T16:34:41Z'
  steps:
    - container: step-say-hello
      imageID: >-
        registry.redhat.io/ubi7/ubi-minimal@sha256:700ec6f27ae8380ca1a3fcab19b5630d5af397c980628fa1a207bf9704d88eb0
      name: say-hello
      terminated:
        containerID: cri-o://346b671912a63a98b310f0f06f0bcd9d9e3fab3b24a75246aed4921863b1d146
        exitCode: 0
        finishedAt: '2022-03-31T16:34:46Z'
        reason: Completed
        startedAt: '2022-03-31T16:34:46Z'
```

OpenShift provides a great dashboard to browse and inspect the Tasks and TasksRun

{:refdef: style="text-align: center;"}
[![](/images/ocp-pipelines/ocp-tasks-dashboard.avif "OpenShift Tasks Dashboard")]({{site.url}}/images/ocp-pipelines/ocp-tasks-dashboard.avif)
{:refdef}

### Pipelines

[`Pipelines`](https://tekton.dev/docs/pipelines/pipelines/) are a collection of `Tasks` that
you define and arrange in a specific order of execution as part of your continuous
integration flow. In fact, tasks should do one single thing so you can reuse them across
pipelines or even within a single pipeline.

You can configure various execution conditions to fit your business needs.

This [example](https://github.com/rmarting/ocp-pipelines-demo/blob/main/05-say-things-in-order-pipeline.yaml) could
give you a general view of a pipeline. This pipeline should be represented as:

{:refdef: style="text-align: center;"}
[![](/images/ocp-pipelines/ocp-pipeline-flow.avif "Pipeline Flow")]({{site.url}}/images/ocp-pipelines/ocp-pipeline-flow.avif)
{:refdef}

Each `Task` in a `Pipeline` executes as a `Pod` on your OpenShift cluster.

Meanwhile a `Pipeline` is a definition, the execution of the pipeline with the results
and outputs is a `PipelineRun`.

OpenShift provides a great dashboard to browse and inspect the Tasks and TasksRun

{:refdef: style="text-align: center;"}
[![](/images/ocp-pipelines/ocp-pipelines-dashboard.avif "OpenShift Pipelines Dashboard")]({{site.url}}/images/ocp-pipelines/ocp-pipelines-dashboard.avif)
{:refdef}

### Workspaces

[`Workspaces`](https://tekton.dev/docs/pipelines/workspaces/) allow `Tasks` to declare parts
of the filesystem that need to be provided at runtime by `TaskRuns`. The main use cases are:

* Storage of inputs and/or outputs
* Sharing data among `Tasks`
* Mount points for configurations held in `Secrets` or `ConfigMaps`
* A cache of build artifacts that speed up jobs

`Workspaces` are similar to `Volumes` except that they allow a `Task` author to defer to
users and their `TaskRuns` when deciding which class of storage to use.

### Triggers

[`Triggers`](https://tekton.dev/docs/triggers/) are the components ready to detect and extract
information from events from a variety of sources and execute `Tasks` or `Pipelines` to respond
them.

Triggers are a set of different objects:

* `EventListener`: listens for events at a specified port on your OpenShift cluster. Specifies
one or more `Triggers` or `TriggerTemplates`.
* `Trigger`: specifies what happens when the `EventListener` detects an event. It is defined
with a `TriggerTemplate`, a `TriggerBinding`, and optionally, an [Interceptor](https://tekton.dev/docs/triggers/interceptors/).
* `TriggerTemplate`: specifies a blueprint for the resource, such as a `TaskRun` or `PipelineRun`, that
you want to instantiate and/or execute when your `EventListener` detects an event.
* `TriggerBinding`: specifies the fields in the event payload from which you want to extract
data and the fields in your corresponding `TriggerTemplate` to populate with the extracted
values. You can then use the populated fields in the `TriggerTemplate` to populate fields in
the associated `TaskRun` or `PipelineRun`.

The most common use case of the `Triggers` and `EventListeners` is integrated with Git repositories
through the use of WebHooks. A Git mechanism to get data from any change in a Git repository.

## Show me the code

But, you could do more things with OpenShift Pipelines, to design you cloud native pipelines. This is a
small briefing of the main characteristics and features. But if you want to play with this new toy, I created
a sample [GitHub repository](https://github.com/rmarting/ocp-pipelines-demo) with a demo of tasks, pipelines and triggers. 

[https://github.com/rmarting/ocp-pipelines-demo](https://github.com/rmarting/ocp-pipelines-demo)

From here, only your imagination, use cases and Tekton could help you to create amazing pipelines in
a easy, descriptive and simple way. 

And if you want to dive deeper, don't miss to check the following references:

* [Tekton Documentation](https://tekton.dev/docs/)
* [Pipelines as Code](https://pipelinesascode.com/), an opinionated CI based on OpenShift Pipelines / Tekton.
* [Tekton Chains](https://github.com/tektoncd/chains) for supply chain security.

Happy cloud-native pipelining :smiley:!!!
