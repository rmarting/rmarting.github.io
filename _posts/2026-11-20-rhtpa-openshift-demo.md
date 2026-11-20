---
layout:      post
title:       ":shield: Exploring Red Hat Trusted Profile Analyzer on OpenShift"
subtitle:    A hands-on demo to deploy Red Hat Trusted Profile Analyzer and Red Hat Dependency Analytics on OpenShift.
description: Overview of the repository to install Red Hat Trusted Profile Analyzer and Red Hat Dependency Analytics on OpenShift.
date:        2026-11-20 09:00:00 +0100
toc:         true
comments:    true
img:         security-logo.avif
fig-caption: Application Security
fig-copy:    true
fig-author:       Pixabay
fig-author-link:  https://www.pexels.com/@pixabay/
fig-gallery:      Pexels
fig-gallery-link: https://www.pexels.com/
tags:
- How-to
- OpenShift
- OpenShift Local
- CRC
- Security
- Red Hat Trusted Profile Analyzer
- Red Hat Dependency Analytics
- SBOM
- Lightwell
- Tutorial
---

Software supply chain security is no longer a nice-to-have. Teams need a clear answer to questions such as:
*What components are in this image? Which CVEs affect my products? How do I share SBOMs and VEX data without
slowing developers down?*

[Red Hat Trusted Profile Analyzer](https://developers.redhat.com/products/trusted-profile-analyzer){:target="_blank" rel="noopener noreferrer"}
(RHTPA) is Red Hat's distribution of the upstream
[Trustify](https://github.com/guacsec/trustify){:target="_blank" rel="noopener noreferrer"} project. It stores and
correlates Software Bills of Materials (SBOMs), advisories, and vulnerability exploitability (VEX) data so platform
and security teams share one system of record.

On the developer side,
[Red Hat Dependency Analytics](https://marketplace.visualstudio.com/items?itemName=redhat.fabric8-analytics){:target="_blank" rel="noopener noreferrer"}
(RHDA) brings that intelligence into the IDE — highlighting risky dependencies while you code, including in AI-assisted
workflows.

I published a demo repository to make the first install on OpenShift as small as possible:

**[github.com/rmarting/rhtpa-openshift-demo](https://github.com/rmarting/rhtpa-openshift-demo){:target="_blank" rel="noopener noreferrer"}**

## 🎯 Goal of the repository

The repository installs **Red Hat Trusted Profile Analyzer** on OpenShift with Kustomize. It also deploys
[Red Hat build of Keycloak](https://www.redhat.com/en/technologies/cloud-computing/openshift/keycloak){:target="_blank" rel="noopener noreferrer"}
(RHBK) so RHTPA has a ready-to-use sign-in experience, plus the PostgreSQL databases both products need.

An later stage adds the **Red Hat Dependency Analytics** backend in the same namespace, so you can connect
your IDE (or CI) to a private RHTPA instance rather than only the public SaaS endpoint.

In short: one repo, two overlays, and a deploy script — enough to go from an empty cluster to a working Trusted Profile
Analyzer console without hand-editing dozens of manifests.

## 🔐 Red Hat Trusted Profile Analyzer and Trustify

RHTPA is the product name; [Trustify](https://github.com/guacsec/trustify){:target="_blank" rel="noopener noreferrer"}
is the open-source engine behind it (part of the GUAC / Trustification community).

At a high level, Trustify / RHTPA helps you:

* Ingest and search **CycloneDX** and **SPDX** SBOMs
* Cross-reference components with advisories from trusted sources (including Red Hat)
* Reduce noise with **VEX** / CSAF exploitability data
* Expose a **REST API** and a web UI for analysts and automation

That combination is what turns "we generate SBOMs somewhere" into "we can answer blast-radius questions when a CVE
lands".

Official docs live under
[Red Hat Trusted Profile Analyzer](https://docs.redhat.com/en/documentation/red_hat_trusted_profile_analyzer/){:target="_blank" rel="noopener noreferrer"}.

## 🧩 Red Hat Dependency Analytics

[Red Hat Dependency Analytics](https://marketplace.visualstudio.com/items?itemName=redhat.fabric8-analytics){:target="_blank" rel="noopener noreferrer"}
is the developer-facing companion. From VS Code (and related surfaces) it analyzes manifests such as `pom.xml`,
`package.json`, `go.mod`, or `requirements.txt`, surfaces known vulnerabilities, and points to remediation.

RHDA matters even more as AI coding assistants generate or suggest dependencies: you want supply-chain checks **in the
editor**, not only after the image is built. You can use the hosted RHDA service, or point the extension at a
**private RHTPA** instance — which is exactly the path this demo prepares for.

## 📦 What is in the repository

The layout follows a classic Kustomize pattern:

| Path | Role |
|------|------|
| `base/` | Shared manifests: operators, RHBK (`keycloak-sso`), the `trustify` realm, RHTPA custom resource, databases |
| `crc/` | Overlay for [OpenShift Local](https://developers.redhat.com/products/openshift-local){:target="_blank" rel="noopener noreferrer"} |
| `ocp/` | Overlay for a standard OpenShift cluster |
| `deploy/` | `deploy.sh` helper that selects the overlay and applies it with `oc` |
| `README-crc.md` / `README-ocp.md` | Step-by-step install and teardown guides |

The components the install brings up include:

* RHBK operator and a `keycloak-sso` instance
* A `trustify` realm with public (`frontend`) and confidential (`cli`) clients, plus lab users
* RHTPA operator and a `TrustedProfileAnalyzer` custom resource
* PostgreSQL for Keycloak and RHTPA

When the install finishes you can open the RHTPA console and the Keycloak account / admin endpoints.

You need `cluster-admin`, a pull secret that can reach `registry.redhat.io`, `oc`, and `python3` (the script reads the
operator catalog).

## 🗺️ Choose your cluster

* **OpenShift Local** — laptop demo with `./deploy/deploy.sh crc`. Follow
  [Install on OpenShift Local](https://github.com/rmarting/rhtpa-openshift-demo/blob/main/README-crc.md){:target="_blank" rel="noopener noreferrer"}.
* **OpenShift** — any cluster you administer (for example from the
  [Red Hat Demo Platform](https://demo.redhat.com/){:target="_blank" rel="noopener noreferrer"}). Follow
  [Install on OpenShift](https://github.com/rmarting/rhtpa-openshift-demo/blob/main/README-ocp.md){:target="_blank" rel="noopener noreferrer"}.

Both paths share the same `base` and the same mental model: operators first, then Keycloak, then RHTPA, then optional
RHDA.

## 🤖 Why this matters in an AI-driven security space

AI is accelerating how we write software — and how attackers probe it. IDPs, copilots, and automated pipelines make it
easier to pull in transitive dependencies. The healthy response is not to slow teams down; it is to give them
**trusted data** at the moment they need it:

* RHTPA as the curated store of SBOMs, advisories, and product risk
* RHDA as the early warning system in the IDE and CI

Detection alone is not enough. When a CVE lands in a popular Java or Python library, the default advice is often
"upgrade to the latest". In enterprise environments that upgrade can break certifications, regression suites, release
windows, or production SLAs. That gap — *I know I am vulnerable, but I cannot move the whole stack today* — is where
a new Red Hat and IBM initiative comes in.

### 💡 Lightwell Network: patched open source without the upgrade roulette

[Lightwell](https://www.redhat.com/en/lightwell){:target="_blank" rel="noopener noreferrer"} is a joint Red Hat and
IBM initiative focused on securing the open source software supply chain. Lightwell aims to give you **signed remediations and patched artifacts** you can consume through secured
repositories — next to the public ecosystems you already use.

In practice, that means you can keep building and shipping while still closing the security gap — pulling a patched
library from a Lightwell repository instead of waiting for a disruptive major upgrade that your release train cannot
absorb yet.

The demo already wires RHTPA to those Lightwell security data sources. In the
[`TrustedProfileAnalyzer` custom resource](https://github.com/rmarting/rhtpa-openshift-demo/blob/main/base/40-rhtpa/trustedprofileanalyzer.yaml){:target="_blank" rel="noopener noreferrer"},
the importer module declares Lightwell OSV feeds, for example:

* `redhat-lightwell-osv-dev` — community Lightwell advisories from
  [project-lightwell/lightwell-osv](https://github.com/project-lightwell/lightwell-osv){:target="_blank" rel="noopener noreferrer"}
* `redhat-lightwell-osv-java-remediated` —
  Java remediated data under `packages.redhat.com/lightwell`
* `redhat-lightwell-osv-python-remediated` — the same pattern for Python remediation

Put together:

* **AI** speeds up discovery of vulnerabilities and generation of new dependencies
* **RHTPA + RHDA** tell you what is affected and where
* **Lightwell Network** gives you a secure way to consume patched open source when a standard upgrade is not an option

If you are exploring that intersection of **AI-assisted development** and **software supply chain security**, clone the
demo, bring up RHTPA on your cluster, connect Dependency Analytics, enable the Lightwell importers when you have
credentials, and walk the console with a real SBOM.

Start here:

* Repository: [rmarting/rhtpa-openshift-demo](https://github.com/rmarting/rhtpa-openshift-demo){:target="_blank" rel="noopener noreferrer"}

Comments, ideas, and PRs are welcome — both on the demo and on this post.

## 📚 References

* [Red Hat Trusted Profile Analyzer](https://developers.redhat.com/products/trusted-profile-analyzer){:target="_blank" rel="noopener noreferrer"}
* [Red Hat Dependency Analytics](https://marketplace.visualstudio.com/items?itemName=redhat.fabric8-analytics){:target="_blank" rel="noopener noreferrer"}
* [Lightwell](https://www.redhat.com/en/lightwell){:target="_blank" rel="noopener noreferrer"}
* [Trustify](https://github.com/guacsec/trustify){:target="_blank" rel="noopener noreferrer"}
* [OpenShift Local](https://developers.redhat.com/products/openshift-local){:target="_blank" rel="noopener noreferrer"}

Happy securing the supply chain :smiley:
