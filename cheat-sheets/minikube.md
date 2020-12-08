---
layout:     post
title:      "My own minikube Cheat Sheet"
toc:        true
comments:   true
permalink:  /cheat-sheets/minikube
tags: 
- How-to
- Cheat Sheet
- minikube
- tutorial
---

[minikube](https://github.com/kubernetes/minikube) quickly sets up a local Kubernetes cluster on
your local laptop. 

:warning: If you need an OpenShift cluster, then you need to review my [CodeReady Containers Cheat Sheet](/cheat-sheets/crc)
or my [Minishift Cheat Sheet](/cheat-sheets/minishift).

This cheat sheet includes the most common commands to install, deploy, administrate or
operate your local Kubernetes using minikube.

Enjoy it !!! :cool: :sunglasses:

## Download and Install

minikube binaries are available [here](https://github.com/kubernetes/minikube/releases).

Download the archive for your operating system and extract its contents, copy it to your preferred
location, and add the minikube binary to your ```PATH``` environment variable.

This [page](https://minikube.sigs.k8s.io/docs/start/) includes the main steps to install minikube. 

To check that your installation:

```bash
❯ minikube version
minikube version: v1.15.1
commit: 23f40a012abb52eff365ff99a709501a61ac5876
```

## Resourcing

By default minikube defines very low resources (CPU, memory). You could check them as:

```bash
❯ minikube config view
- cpus: 4
- memory: 16384
```

To assign the number of CPU:

```bash
❯ minikube config set cpus 4
```

To assign the memory:

```bash
❯ minikube config set memory 16384
```

## Start

To start the default, or active profile:

```bash
❯ minikube start
```

To start a profile:

```bash
❯ minikube start --profile myprofile
```

## Endpoints and Credentials

You could get the Kubernetes console as:

```bash
❯ minikube dashboard
```

## Using profiles

When you have to manage different configurations, applications or deployments, you could use different namespaces
in the same minikube instance, or use different **profile**s.

A profile is an instance of the minikube VM along with all of its configuration and state. The profile feature
allows you to create and manage these isolated instances of minikube.

Each profile is created with its own configuration (memory, CPU, disk size, add-ons, and so on) and is
independent of other profiles. 

The active profile is the profile against which all commands are executed, unless the global ```--profile``` flag is
used. 

To create a new profile:

```bash
❯ minikube start -p demo
```

To list the profiles available:

```bash
❯ minikube profile list
|----------|-----------|---------|----------------|------|---------|---------|
| Profile  | VM Driver | Runtime |       IP       | Port | Version | Status  |
|----------|-----------|---------|----------------|------|---------|---------|
| demo     | kvm2      | docker  |                | 8443 | v1.19.2 | Running |
| minikube | kvm2      | docker  | 192.168.39.213 | 8443 | v1.19.2 | Stopped |
|----------|-----------|---------|----------------|------|---------|---------|
```

To delete a profile:

```bash
❯ minikube delete -p demo
```

## Addons

minikube allows you to extend the Kubernetes setup with an add-on mechanism.

minikube provides a set of built-in add-ons that offer some common Kubernetes customization to assist
with development. To list the default add-ons, run:

```bash
❯ minikube addons list
|-----------------------------|----------|--------------|
|         ADDON NAME          | PROFILE  |    STATUS    |
|-----------------------------|----------|--------------|
| ambassador                  | minikube | disabled     |
| csi-hostpath-driver         | minikube | disabled     |
| dashboard                   | minikube | enabled ✅   |
| default-storageclass        | minikube | enabled ✅   |
| efk                         | minikube | disabled     |
| freshpod                    | minikube | disabled     |
| gcp-auth                    | minikube | disabled     |
| gvisor                      | minikube | disabled     |
| helm-tiller                 | minikube | disabled     |
| ingress                     | minikube | enabled ✅   |
| ingress-dns                 | minikube | disabled     |
| istio                       | minikube | disabled     |
| istio-provisioner           | minikube | disabled     |
| kubevirt                    | minikube | disabled     |
| logviewer                   | minikube | disabled     |
| metallb                     | minikube | disabled     |
| metrics-server              | minikube | disabled     |
| nvidia-driver-installer     | minikube | disabled     |
| nvidia-gpu-device-plugin    | minikube | disabled     |
| olm                         | minikube | enabled ✅   |
| pod-security-policy         | minikube | disabled     |
| registry                    | minikube | enabled ✅   |
| registry-aliases            | minikube | disabled     |
| registry-creds              | minikube | disabled     |
| storage-provisioner         | minikube | enabled ✅   |
| storage-provisioner-gluster | minikube | disabled     |
| volumesnapshots             | minikube | disabled     |
|-----------------------------|----------|--------------|
```

To enable and apply an add-on:

```bash
❯ minikube addons enable olm
```

To disable an addon:

```bash
❯ minikube addons disable kubevirt
```

## Status

To check the status of your minikube:

```bash
❯ minikube status -p demo
demo
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

## Stop

To stop your minikube instance:

```bash
❯ minikube stop
```

## Delete

Deleting the minikube instance you will recover the storage space of your laptop or desktop computer:

```bash
❯ minikube delete
```

:warning: This operation could not be recovered. :warning:

## Updating minikube

To check if there is new updated versions:

```bash
❯ minikube update-check
CurrentVersion: v1.14.0
LatestVersion: v1.15.1
```
