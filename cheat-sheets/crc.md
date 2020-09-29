---
layout:     page
title:      "My own CodyReady Containers Cheat Sheet"
toc:        true
permalink:  /cheat-sheets/crc
date:       2020-10-29 10:00
published:  true
status:     publish
categories: 
- how-to
- github-pages
- jekyll
tags: []
author:     rmarting
comments:   true
---

[Red Hat CodeReady Container](https://developers.redhat.com/products/codeready-containers/overview) (CRC to abbreviate)
brings a minimal, preconfigured OpenShift 4 cluster :cloud: to your local laptop or desktop computer for development and testing purposes.

This cheat sheet includes the most commond commands to install, deploy, administrate or
operate your local OpenShift using CRC.

Enjoy it !!! :cool: :sunglasses:

## Download and Install

CRC will need an local virtual machine image (ISO) and a
valid pull secret to download images from [Red Hat Container Registry](registry.redhat.io).

Both things are available [here](https://cloud.redhat.com/openshift/install/crc/installer-provisioned)

Run the this command command to set up your host operating system.

```bash
❯ crc setup
```

## Start

The easy way to start is passing the reference to the pull secret, it will avoid asking you
anything on startup time:

```bash
❯ crc start -p /path/to/pull-secret.json
```

## Endpoints and Credentials

Your new local OCP 4 cluster is available here:

* API (to be used with ```oc``` command): https://api.crc.testing:6443
* Web Console: https://console-openshift-console.apps-crc.testing

You could check these endpoints with:

```bash
❯ crc console --url
```

Users provided are:

* ```kubeadmin``` as *cluster-admin*
* ```developer``` as regular user

To get the current credentials:

```bash
❯ crc console --credentials
To login as a regular user, run 'oc login -u developer -p developer https://api.crc.testing:6443'.
To login as an admin, run 'oc login -u kubeadmin -p duduw-yPT9Z-hsUpq-f3pre https://api.crc.testing:6443'
```

## Define an user as cluster-admin

Maybe you want to use other different user as *cluster-admin* instead of kubeadmin. This
command will promote to *cluster-admin* the *developer* user.

```bash
❯ oc adm policy add-cluster-role-to-user cluster-admin developer
```

:warning: A high power requires higher responsability. :warning:

## Create new users

CRC provides a small set of users:

* ```kube-admin``` as *cluster-admin*
* ```developer``` as regular user

To create new users, we must export the **htpass-secret** used by OCP to authenticated them and declare there
new users.

* Extract htpasswd secret:

```bash
❯ oc get secret htpass-secret -ojsonpath={.data.htpasswd} -n openshift-config | base64 -d > users-crc.htpasswd
```

* Add/Update/Delete users in ```users-crc.htpasswd```:

- Add or Update: ```htpasswd -bB users-crc.htpasswd <username> <password>```
- Delete: ```htpasswd -D users-crc.htpasswd <username>```

* Update htpasswd secret:

```bash
❯ oc create secret generic htpass-secret --from-file=htpasswd=users-crc.htpasswd \
  --dry-run=client -o yaml -n openshift-config | oc replace -f -
```

Now you have a large list of users to use.

References: 

* [OpenShift - Updating users for an HTPasswd identity provider](https://docs.openshift.com/container-platform/4.5/authentication/identity_providers/configuring-htpasswd-identity-provider.html#identity-provider-htpasswd-update-users_configuring-htpasswd-identity-provider)

## Status

To check the status of your CRC:

```bash
❯ crc status
CRC VM:          Running
OpenShift:       Running (v4.5.9)
Disk Usage:      22.92GB of 32.72GB (Inside the CRC VM)
Cache Usage:     37.3GB
Cache Directory: /home/rmarting/.crc/cache
```

## Stop

To stop your CRC instance:

```bash
❯ crc stop
```

## Delete

Deleting the CRC instance you will recover the storage space of your laptop or desktop computer:

```bash
❯ crc delete
```

:warning: This operation could not be recovered. :warning:
