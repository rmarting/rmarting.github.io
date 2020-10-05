---
layout:     post
title:      "My own MiniShift Cheat Sheet"
toc:        true
comments:   true
permalink:  /cheat-sheets/minishift
tags: 
- How-to
- Cheat Sheet
- minishift
- tutorial
---

[Minishift](https://docs.okd.io/3.11/minishift/getting-started/index.html) is a tool that helps you
run OpenShift 3 locally by running a single-node OpenShift cluster inside a VM. You can try out
OpenShift or develop with it, day-to-day, on your local host.

:warning: If you need an OpenShift 4 cluster, then you need to review my [CodeReady Containers Cheat Sheet](/cheat-sheets/crc).

:point_right: [Red Hat Container Development Kit](https://developers.redhat.com/products/cdk/overview) is the enterprise
version of minishift provided by Red Hat.

This cheat sheet includes the most commond commands to install, deploy, administrate or
operate your local OpenShift using minishift.

Enjoy it !!! :cool: :sunglasses:

## Download and Install

minishift binaries are available [here](https://github.com/minishift/minishift/releases).

Download the archive for your operating system and extract its contents, copy it to your preferred
location, and add the minishift binary to your ```PATH``` environment variable.

To check that your installation:

```bash
❯ minishift version
minishift v1.34.2+c50597d
```

:warning: CDK version requires run the this command to set up your host operating system before
to start it.

```bash
❯ minishift setup-cdk
```

## Resourcing

By default minishift defines very low resources (cpus, memory). You could check them as:

```bash
❯ minishift config view
- iso-url                            : file:///home/rmarting/.minishift/cache/iso/minishift-rhel7.iso
- memory                             : 4096
- vm-driver                          : kvm
```

To assing the number of cpus:

```bash
❯ minishift config set cpus 4
```

To assing the memory:

```bash
❯ minishift config set memory 16384
```

## Start

To start the default, or active profile:

```bash
❯ minishift start
```

To start a profile:

```bash
❯ minishift start --profile myprofile
```

:raising_hand: CDK needs to register with Red Hat to download and build container images. Add your
Red Hat Developer Program username to your environment to avoid the need to enter it each
time you start minishift. Your password is stored in the system keystore, so after you log in once,
the CDK simply retrieves the password from the keystore.

```bash
❯ export MINISHIFT_USERNAME='<RED_HAT_USERNAME>'
❯ export MINISHIFT_PASSWORD='<RED_HAT_PASSWORD>'
```

## Endpoints and Credentials

Every time minishift is started the IP address may change, that information is provided
in the output log:

```bash
❯ minishift start
-- Starting profile 'minishift'
...
   OpenShift server started.
   The server is accessible via web console at:
       https://192.168.42.52:8443

   You are logged in as:
       User:     developer
       Password: developer

   To login as administrator:
       oc login -u system:admin
```

You could get the OpenShift console as:

```bash
❯ minishift console --url
```

Users provided are:

* ```system:admin``` as *cluster-admin* (not recommended to use, unless you need to administrate your own OpenShift cluster)
* ```developer``` as regular user (recommended user). This user also can impersonate the administrator. This
allows you to run administrator commands using the ```--as system:admin``` parameter.

To log in as *developer*:

```bash
❯ oc login -u developer -p developer https://$(minishift ip):8443
```

To log in as *cluster-admin*:

```bash
❯ oc login -u system:admin https://$(minishift ip):8443
```

## Using profiles

When you have to manage different configurations, applications or deployments, you could use different namespaces
in the same minishift instance, or use different **profile**s.

A profile is an instance of the Minishift VM along with all of its configuration and state. The profile feature
allows you to create and manage these isolated instances of Minishift.

:warning: This feature is only available in CDK. :warning: 

Each CDK profile is created with its own configuration (memory, CPU, disk size, add-ons, and so on) and is
independent of other profiles. 

The active profile is the profile against which all commands are executed, unless the global ```--profile``` flag is
used. 

To create a new profile:

```bash
❯ minishift profile set myprofile
Setting up CDK 3 on host using '/home/rmarting/.minishift/profiles/myprofile' as Minishift's home directory
Creating configuration file '/home/rmarting/.minishift/profiles/myprofile/config/config.json'
Creating marker file '/home/rmarting/.minishift/profiles/myprofile/cdk'
Default add-ons anyuid, admin-user, xpaas, registry-route, che, htpasswd-identity-provider, eap-cd installed
Default add-ons anyuid, admin-user, xpaas enabled
Profile 'myprofile' set as active profile.
```

To delete a profile:

```bash
❯ minishift profile delete myprofile
Will remove the VM and all the related artifacts. Do you want to continue [y/N]?: y
Profile 'myprofile' deleted successfully.
```

## Addons

Minishift allows you to extend the OpenShift setup with an add-on mechanism.

Minishift provides a set of built-in add-ons that offer some common OpenShift customization to assist
with development. To list the default add-ons, run:

```bash
❯ minishift addons list
- admin-user                  : enabled	P(0)
- anyuid                      : enabled	P(0)
- xpaas                       : enabled	P(0)
- che                         : disabled	P(0)
- eap-cd                      : disabled	P(0)
- htpasswd-identity-provider  : disabled	P(0)
- registry-route              : disabled	P(0)
```

To enable and apply an add-on:

```bash
❯ minishift addons enable registry-route
❯ minishift addons apply registry-route
```

CDK enables automatically a set of add-ons: ```xpass```, ```admin-user``` and ```anyuid```.

```log
-- Applying addon 'xpaas':..
XPaaS imagestream and templates for OpenShift installed
See https://github.com/openshift/openshift-ansible/tree/release-3.11/roles/openshift_examples/files/examples
-- Applying addon 'admin-user':..
-- Applying addon 'anyuid':.
 Add-on 'anyuid' changed the default security context constraints to allow pods to run as any user.
 Per default OpenShift runs containers using an arbitrarily assigned user ID.
 Refer to https://docs.okd.io/latest/architecture/additional_concepts/authorization.html#security-context-constraints and
 https://docs.okd.io/latest/creating_images/guidelines.html#openshift-origin-specific-guidelines for more information.
```

## Define an user as cluster-admin

The easy way to define a *cluster-admin* user is using the **admin-user** add-on:

```bash
❯ minishift addons enable admin-user
Add-on 'admin-user' enabled
❯ minishift addons apply admin-user
```

This add-on will create a new user identified by **admin** (same password):

```bash
❯ oc login -u admin -p admin https://$(minishift ip):8443
```

Maybe you want to use other different user as *cluster-admin*. This
command will promote to *cluster-admin* the *developer* user.

```bash
❯ oc login -u system:admin
❯ oc adm policy add-cluster-role-to-user cluster-admin developer
```

:warning: A high power requires higher responsability. :warning:

## Status

To check the status of your minishift:

```bash
❯ minishift status
Minishift:  Running
Profile:    minishift
OpenShift:  Running (openshift v3.11.232)
DiskUsage:  24% of 19G (Mounted On: /mnt/sda1)
CacheUsage: 1.446 GB (used by oc binary, ISO or cached images)
```

## Stop

To stop your minishift instance:

```bash
❯ minishift stop
```

## Delete

Deleting the minishift instance you will recover the storage space of your laptop or desktop computer:

```bash
❯ minishift delete
```

:warning: This operation could not be recovered. :warning:

## Updating Minishift

:warning: This option is not valid for the CDK version :warning:

To update minishift:

```bash
❯ minishift update
```

This command checks whether there is a newer version of Minishift available. If so, it prompts user
to confirm the update, downloads the new binary and replaces the current version of Minishift.

## Updating CDK

CDK has not an *update* automatic method, so you have to follow the next steps:

1. Download new binary
2. Delete your current profiles
3. Setup CDK
4. Create or define your profiles
5. Start each profile

I know that it is not so good ... but this is the way!

## Register images to registry.redhat.io

Red Hat Registry is an authenticated service, so it is needed to have valid
credentials to pull images from here.

Using a Red Hat account create a new token in
[Registry Service Accounts](https://access.redhat.com/terms-based-registry). Registry Service
Accounts are named tokens that can be used in environments where credentials will
be shared, such as deployment systems.

This tool will create a secret (in yaml format) with the valid credentials to pull images. Download
the secret yaml from the *OpenShift Secret* tab.

This secret will be similar to:

```yaml
❯ more cdk-3-secret.yaml 
apiVersion: v1
kind: Secret
metadata:
  name: cdk-3-pull-secret
data:
  .dockerconfigjson: **************
type: kubernetes.io/dockerconfigjson
```

Create the secret in the namespace:

```bash
❯ oc create -f cdk-3-secret.yaml
```

Link secret to *default* and *builder* service accounts.

```bash
❯ oc secrets link default cdk-3-pull-secret --for=pull
❯ oc secrets link builder cdk-3-pull-secret
```

## Using Wildcard Routes (for a Subdomain)

OpenShift 3 has support for wildcard routes, however this feature is not
enabled by default. This feature could be needed for some deployments so
the HAProxy should be modified to allow them.

This must be executed with a *cluster-admin* user.

Using your **developer** user you could execute the following commands impersonate as *cluster-admin*:

```bash
❯ oc scale dc/router --replicas=0 -n default --as=system:admin
deploymentconfig.apps.openshift.io/router scaled
❯ oc set env dc/router ROUTER_ALLOW_WILDCARD_ROUTES=true -n default --as=system:admin
deploymentconfig.apps.openshift.io/router updated
❯ oc scale dc/router --replicas=1 -n default --as=system:admin
deploymentconfig.apps.openshift.io/router scaled
```

References:

* [Using Wildcard Routes](https://docs.openshift.com/container-platform/3.11/install_config/router/default_haproxy_router.html#using-wildcard-routes)

## Throubleshooting

### minishift_kubeconfig does not exists

Sometimes when minishift starts could fail with the next error message:

```log
Could not set oc CLI context for 'minishift' profile: Error during setting 'minishift' as active profile: The specified path to the kube config '/home/rmarting/.minishift/machines/minishift_kubeconfig' does not exist
```

or this other one:

```log
Error applying the add-on: The specified path to the kube config '/home/rmarting/.minishift/machines/minishift_kubeconfig' does not exist
```

I don't know the root cause of this issue, however create a symbolic link as workaround works for me: 

```bash
❯ ln -s ~/.kube/config ~.minishift/machines/minishift_kubeconfig
```

:point_right: This issue is tracked in [CDK-360](https://issues.redhat.com/browse/CDK-360) issue. 
