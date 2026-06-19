---
layout:      post
title:       ":rocket: From empty cluster to Internal Developer Portal"
subtitle:    A hands-on Red Hat Developer Hub workshop for platform engineers on OpenShift.
description: Summary of a full-day Red Hat Developer Hub workshop covering GitLab integration, catalog, RBAC, templates, TechDocs, and advanced IDP capabilities on OpenShift.
date:        2026-06-19 09:00:00 +0200
toc:         true
comments:    true
img:         how-to.avif
fig-caption: Developer Hub
fig-copy:    true
fig-author:       Pixabay
fig-author-link:  https://www.pexels.com/@pixabay
fig-gallery:      Pexels
fig-gallery-link: https://www.pexels.com/
tags:
- How-to
- OpenShift
- Red Hat Developer Hub
- GitLab
- Backstage
- Tutorial
- Community
- DevEx
- Platform Engineering
---

Your teams already use a SCM. Repositories exist. Pipelines run. But developers still ask the same questions:
*Where is that service? Who owns it? How do I scaffold a new component? Where is the documentation?*

An internal developer portal (IDP) answers those questions in one place.
[Red Hat Developer Hub](https://developers.redhat.com/products/rhdh){:target="_blank" rel="noopener noreferrer"} brings that experience to OpenShift with a
Backstage-based portal, dynamic plugins, and enterprise-ready operations. The challenge for platform engineers is
not deciding *whether* to adopt an IDP — it is knowing *where to start* when the product spans authentication,
catalog discovery, RBAC, plugins, TechDocs, workflows, and AI integrations.

I built a hands-on workshop to walk through that journey step by step. This post summarizes what the lab
covers, who it is for, and what you walk away with at the end.

## 🦊 The scenario: GitLab without a portal

The workshop puts you in the role of a platform engineer evaluating Red Hat Developer Hub for your organization.
Using GitLab as SCM the onboarding is manual. There is no central catalog, no unified sign-in
aligned with GitLab identity, no standard way to scaffold components, and no governance over self-service actions.

Your assignment: install Red Hat Developer Hub, integrate it with GitLab, and enable the capabilities your
developers need day to day.

By the end, you have a working portal where users sign in with GitLab, the catalog discovers repositories and
teams automatically, and extended capabilities with templates, documentation, automation, and
CI/CD visibility.

## 📋 What the workshop covers

The content is organized as a Showroom lab — incremental modules with apply-ready manifests and `oc`
commands. Each step builds on the previous one; nothing is a disconnected demo.

### ⚙️ Foundation: Operators

You start on an operational and empty [OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift){:target="_blank" rel="noopener noreferrer"}
to set up with the operators needed, and services to build the final solution.

### 🔗 GitLab integration

Four modules cover GitLab end to end:

1. **Authentication** — GitLab OAuth sign-in so portal users match GitLab identity.
2. **Plugin integration** — Dynamic plugins for GitLab frontend and backend, loaded from OCI overlays.
3. **Catalog autodiscovery** — GitLab projects with `catalog-info.yaml` appear in the software catalog automatically.
4. **Users and groups** — GitLab org data mirrored as catalog entities via the org provider.

### 🛡️ Governance and self-service

**RBAC** introduces role-based access control with policies and conditional rules. You validate that different
GitLab users see different capabilities — essential before opening self-service templates to a wide audience.

**Software templates** register scaffolder templates from a Git location and walk through creating a new component
from the portal UI. This is often the moment developers realize the IDP is more than a catalog browser.

### 🧩 Extending the portal

**Dynamic plugins** show how to deploy custom or community plugins without rebuilding the hub image — core to how
Red Hat Developer Hub evolves faster than a monolithic Backstage deployment.

**TechDocs** configures documentation publishing with object storage, GitLab pipeline
integration, and external builder/generator settings so docs live beside the components they describe.

**Extended features** round out day-two operations:

**High availability** for resilience 

**Dynamic plugins cache** for fast start up

**Monitoring and Adoption Insights** to track the portal usage by users

**Notifications** to be aware of internal events

### 🚀 Advanced capabilities

**Orchestrator** installs SonataFlow-based workflows you trigger from the portal — useful for approvals, provisioning
sequences, or any multi-step automation your platform team wants to expose safely.

**AI integrations** cover two areas:

* **Model Context Protocol (MCP)** — expose catalog and TechDocs to AI clients.
* **Red Hat Developer Lightspeed** — conversational assistance grounded in product documentation.

**Enterprise extensions** close the day with patterns teams need at scale:

* External PostgreSQL instead of the operator-managed local database
* CI/CD visibility through Tekton, Kubernetes, and Argo CD dynamic plugins on entity pages

## 👥 Who should take this workshop

The content fits:

* **Platform engineers** installing and operating Red Hat Developer Hub on OpenShift
* **DevEx teams** evaluating internal developer portals against GitLab-centric workflows
* **Developers** who want hands-on experience with catalog, templates, TechDocs, and plugins — not just a product
  slide deck

If you already know Backstage concepts, the lab focuses on Red Hat Developer Hub-specific operator patterns,
dynamic plugin packaging, and Red Hat documentation.

## 🎁 What you leave with

After it, you have:

* A GitLab-authenticated Red Hat Developer Hub instance on OpenShift
* Catalog entities discovered from GitLab projects, users, and groups
* RBAC policies enforcing team-scoped access
* Software templates registered and exercised end to end
* Dynamic plugins, TechDocs, HA, monitoring, and notifications configured
* Exposure to Orchestrator, AI capabilities, external database, and CI/CD plugins

That is a credible starting point for a production IDP — not a throwaway sandbox..

## 🧪 Try it yourself

The workshop runs as an interactive Showroom lab:

* [Red Hat Developer Hub Workshop — Showroom](http://blog.jromanmartin.io/rhdh-exercises/){:target="_blank" rel="noopener noreferrer"}
* Source repository: [github.com/rmarting/rhdh-exercises](https://github.com/rmarting/rhdh-exercises){:target="_blank" rel="noopener noreferrer"}

Clone the repo, order an OpenShift cluster, and follow the modules from Environment Setup through Wrap up.
Each section includes verification steps so you know the configuration landed before moving on.

## 📚 References

* [Red Hat Developer Hub product page](https://developers.redhat.com/products/rhdh){:target="_blank" rel="noopener noreferrer"}
* [Red Hat Developer Hub documentation](https://docs.redhat.com/en/documentation/red_hat_developer_hub/){:target="_blank" rel="noopener noreferrer"}
* [Workshop source — rhdh-exercises](https://github.com/rmarting/rhdh-exercises){:target="_blank" rel="noopener noreferrer"}

Happy platform engineering and developer experience :smiley:
