---
layout:      post
title:       ":lock: Trust the OpenShift Local Root CA on your workstation"
subtitle:    Export and install the OpenShift Local ingress root CA so routes and APIs validate as trusted.
description: How to export the OpenShift Local root CA and add it to your local trust store for development and testing.
date:        2026-04-20 09:00:00 +0200
toc:         true
comments:    true
img:         account-verified.avif
fig-caption: OpenShift Local
fig-copy:    true
fig-author:       Zulfugar Karimov
fig-author-link:  https://www.pexels.com/@zulfugarkarimov/
fig-gallery:      Pexels
fig-gallery-link: https://www.pexels.com/
tags:
- How-to
- OpenShift
- OpenShift Local
- CRC
- tutorial
- security
---

[OpenShift Local](https://developers.redhat.com/products/openshift-local)runs a minimal
[OpenShift Container Platform](https://www.redhat.com/en/technologies/cloud-computing/openshift) cluster on your laptop.
Routes exposed under the default `*.apps-crc.testing` domain are secured with certificates signed by a
**cluster-local root CA** created by the [Ingress Operator](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/networking_operators/configuring-ingress).

That CA is trusted inside the cluster, but **not** on your workstation. As a result, you may see:

* Browser warnings when opening the [web console](https://console-openshift-console.apps-crc.testing) or application routes.
* TLS errors in tools such as `curl`, language HTTP clients, or IDE extensions that call cluster endpoints.
* Failed certificate validation when testing OAuth flows, webhooks, or any integration that expects a publicly trusted chain.

This post shows how to export that root CA and install it in your local trust store so OpenShift Local endpoints behave
like they are signed by a trusted authority — which is especially useful when you need to verify services as
**official** and **trusted** during local development.

:information_source: This approach is intended for **local development and testing only**. Do not import development CAs
into production systems or shared machines.

## Prerequisites

Before you start, ensure that:

* [OpenShift Local is installed and running](https://developers.redhat.com/products/openshift-local/getting-started).
  You can confirm with `crc status`.
* You can reach the default console URL:
  [https://console-openshift-console.apps-crc.testing](https://console-openshift-console.apps-crc.testing)
* For the recommended extraction method, the `oc` CLI is available and you are logged in to the cluster
  (for example, with `crc console --credentials`).

If you are new to OpenShift Local, my [OpenShift Local Cheat Sheet](/cheat-sheets/crc) covers installation,
endpoints, and common operations.

## Why a custom root CA exists

By default, OpenShift uses the Ingress Operator to create an internal CA and issue a wildcard certificate for the
`*.apps` subdomain. As described in the
[OpenShift certificate documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/security_and_compliance/configuring-certificates#replacing-default-ingress),
these infrastructure certificates are **self-signed**. Components inside the cluster trust them automatically; external
clients on your laptop do not.

The signing certificate is stored in the `router-ca` secret in the `openshift-ingress-operator` namespace. Exporting
that certificate and trusting it locally closes the gap between in-cluster and workstation trust.

## Export the root CA

Log in to the cluster if you have not already, then run:

```bash
openssl s_client -showcerts -verify 5 -connect console-openshift-console.apps-crc.testing:443 < /dev/null 2>&1 | \
sed -n '/-----BEGIN CERTIFICATE-----/{:a; N; /-----END CERTIFICATE-----/!b a; h}; ${g;p}' > crc-root-ca.pem
```

(Optional) Other alternative is to get the last certificate of the chain with this command:

```bash
openssl s_client -showcerts -verify 5 -connect console-openshift-console.apps-crc.testing:443 < /dev/null 2>&1 | \
sed -n '/-----BEGIN CERTIFICATE-----/{:a; N; /-----END CERTIFICATE-----/!b a; h}; ${g;p}' > crc-root-ca.pem
```

Verify the Root CA:

```bash
openssl x509 -in crc-root-ca.pem -noout -subject -issuer
```

For the default OpenShift Local ingress CA, **subject** and **issuer** should match, which confirms it is a
self-signed root certificate. The output looks similar to:

```text
subject=CN=ingress-operator@1758703981
issuer=CN=ingress-operator@1758703981
```

## Import the root CA into your local workstation

The exact commands depend on your operating system. In all cases, you are adding `crc-root-ca.pem` as an **anchor**
(a trusted root), not replacing system CAs.

These distributions use the [`update-ca-trust`](https://docs.fedoraproject.org/en-US/quick-docs/using-shared-system-certificates/) framework. 

Don't forget to restart browsers and any long-running applications so they reload the system trust store.

### Fedora

```bash
sudo cp crc-root-ca.pem /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust
```

### Debian and Ubuntu

```bash
sudo cp crc-root-ca.pem /usr/local/share/ca-certificates/crc-root-ca.crt
sudo update-ca-certificates
```

Note the `.crt` extension, which `update-ca-certificates` expects.

### macOS

```bash
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain crc-root-ca.pem
```

You may need to approve the change in **Keychain Access**.

### Windows

1. Open `certlm.msc` (Local Computer certificate manager).
2. Navigate to **Trusted Root Certification Authorities → Certificates**.
3. Right-click **Certificates → All Tasks → Import**.
4. Select `crc-root-ca.pem` and complete the wizard.
5. Restart applications that cache TLS trust.

## Confirm that trust works

After importing the CA, verify that TLS validation succeeds **without** skipping certificate checks.

Using `curl`:

```bash
curl -v https://console-openshift-console.apps-crc.testing 2>&1 | grep -E 'SSL certificate verify|subject:'
```

You should not see certificate verification errors. Opening the console URL in a browser should no longer show an
untrusted certificate warning.

If validation still fails:

* Confirm OpenShift Local is running (`crc status`).
* Ensure you imported the **root** CA, not a leaf route certificate.
* Restart the client application; some tools cache trust stores at startup.
* On Linux, run `trust list | grep -i ingress` to confirm the anchor is present.

## When you might need this

Trusting the OpenShift Local root CA is useful when you want realistic TLS behavior during development, for example:

* Testing browser-based login flows against routes without accepting security exceptions.
* Calling HTTPS APIs from Postman, VS Code extensions, or automated test suites.
* Validating mTLS or OAuth redirect URLs that require a trusted chain.
* Demonstrating integrations where partners or tools reject self-signed certificates.

For day-to-day cluster administration, see the [OpenShift Local documentation](https://crc.dev/docs)
and the [ingress certificate types reference](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/security_and_compliance/certificate-types-and-descriptions#cert-types-ingress-certificates).

## References

* [Red Hat OpenShift Local — Getting started](https://developers.redhat.com/products/openshift-local/getting-started)
* [OpenShift Local documentation](https://crc.dev/docs)
* [Replacing the default ingress certificate](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/security_and_compliance/configuring-certificates#replacing-default-ingress)
* [Ingress certificates — certificate types and descriptions](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/security_and_compliance/certificate-types-and-descriptions#cert-types-ingress-certificates)
* [CodeReady Containers Cheat Sheet](/cheat-sheets/crc)
* [CRC issue #2820 — Export router CA certificate](https://github.com/crc-org/crc/issues/2820)

Happy local OpenShift developing :smiley:
