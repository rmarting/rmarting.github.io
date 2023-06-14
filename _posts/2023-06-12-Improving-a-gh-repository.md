---
layout:      post
title:       "📛 Improving a GitHub Repo!"
description: Improve a GitHub repo with badges and automatic releasing!!!
date:        2023-06-12 09:00:00 +0200
toc:         true
comments:    true
img:         computer-and-notes.avif
fig-caption: GitHub
fig-copy:    true
fig-author:       Lukas
fig-author-link:  https://www.pexels.com/@goumbik/
fig-gallery:      Pexels
fig-gallery-link: https://www.pexels.com/
tags:
- Community
- GitHub
- git
- productivity
- How-to
- Tutorial
- development
- CICD
- tools
---

I have been using [GitHub](https://github.com) for a long time and I spent time on a daily
basis reviewing repos in the Open Source space. One of the most important things
, from my point of view, is to get a good overview of the repository, a good documentation,
but also good highlights, such as releases, status of the project, Changelogs, Contribution
Guides, emojis ([why not?](https://blog.jromanmartin.io/2020/09/28/why-i-use-emoji-in-my-git-commits.html)) ...
so I can get faster a good summary of the repository. This is not easy and
there are many different ways to do it, but I found some of them very easy to add in any repository.

This post covers two of these mechanisms to improve any GitHub Repository:

* 📛 Repository Badges
* ✅ Changelogs and 🤖🚩automatic releasing process

## 📛 Repository Badges

How does it look like a repo with badges? Something like this:

{:refdef: style="text-align: center;"}
[![](/images/2023/06/github/gh-repo-badges.avif "GitHub repo with badges")]({{site.url}}/images/2023/06/github/gh-repo-badges.avif)
{: refdef}

Nice 🫶, right?

Badges are an easy way to summarize a repo with information about topics such
as building, test results, license, pipelines or workflows, versions, ... This information
provides quality metadata coming from many different resources. So meanwhile you are browsing,
you get all this information in a single view. Incredible!

I found a simple way to integrate almost any badge in my repository ...
[Shields.io](https://shields.io/). It is a service providing badges in different formats to
integrate in GitHub readme files. This service supports a bunch of continuous integration
services, package registries, distributions, app stores, social networks, code coverage services,
and code analysis services (anything else? 🤷🏽‍♀️).

In short, using the web site you can customize your badge to your own requisites and 3rd party
services, getting a code to add in your GitHub readme easily.

For example, the previous image is rendered using the next entries in my readme file of my
Blog site repository:

```markdown
![License](https://img.shields.io/github/license/rmarting/rmarting.github.io?style=plastic)
![Main Lang](https://img.shields.io/github/languages/top/rmarting/rmarting.github.io)
![Languages](https://img.shields.io/github/languages/count/rmarting/rmarting.github.io)
![Last Commit](https://img.shields.io/github/last-commit/rmarting/rmarting.github.io)
```

Easy ✅, and powerful 💪! So, don't forget to add your badges in your repo to help me, and
others 🤗!

## ✅ Changelogs and 🤖🚩automatic releasing process

Changelog, as a comprehensive and up-to-date file, is crucial for effective
project management and collaboration. A changelog serves as a documented record
of all the notable changes, enhancements, and bug fixes made to your software
over time. It not only provides transparency and accountability but also facilitates
communication among team members and external contributors. This file enables users
and developers to easily track the evolution of the project, understand the latest
features and improvements, and quickly identify any potential issues or compatibility concerns.

Getting all these benefits require a regular updating of that file, usually after releasing a new
version or iteration of our software. But, how to track all the changes between versions? Who
should do it? When? ... It seems that it could be tedious every time if we have to do it manually
... we can forget something to add, or we can forget to update it at all.

As fan of ...

{:refdef: style="text-align: center;"}
[![](/images/2023/06/github/automate-all-the-things.avif "Automate all the things (Sticker)")](https://www.redbubble.com/i/sticker/AUTOMATE-ALL-THE-THINGS-by-antonwadstrom/29760692.EJUG5)
{: refdef}

There is a way to automatically update the changelog file every time a new version is released.
This blog post summarizes this process.

**Step 1️⃣ - Create your Changelog file**

Create a file, usually called `CHANGELOG.md`, with the following content:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
```

To delve deeper into the significance of changelog files and learn about best practices for
creating and maintaining them, I recommend checking out [Keep a Changelog](https://keepachangelog.com/).
This resource offers a comprehensive guide and industry-accepted standards for crafting informative
and well-structured changelogs.

**Step 2️⃣ - Use a Release workflow to publish new releases**

The [Release Drafter GitHub Action](https://github.com/marketplace/actions/release-drafter) is an
incredible GitHub action to automate a new release of the repository. The action is initially designed
to draft a new release, but it is also valid to release automatically the version. In my case,
I will automatically release the version as soon as a new tag is pushed.

The following `release-drafter.yml` file inside of the `.github/workflows` folder will publish a new
release after a new tag is pushed into the repository. The tag must be aligned with the
[Semantic Versioning](https://semver.org):

```yaml
name: 🔖 Release Drafter 🔖

on:
  push:
    tags:
      - v[0-9]+.[0-9]+.[0-9]+

permissions:
  contents: read
      
jobs:
  update_release_draft:
    name: Release drafter
    runs-on: ubuntu-latest
    permissions:
      # write permission is required to create a github release
      contents: write

    steps:
      - name: Update Release Draft
        uses: release-drafter/release-drafter@v5
        with:
          publish: true
          prerelease: false
        env:
          # Instead of GITHUB_TOKEN Ref: https://github.com/stefanzweifel/changelog-updater-action/discussions/30
          GITHUB_TOKEN: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
```

The `publish: true` attribute publishes the release as final, just because the `prerelease` attribute
is marked as `false`.

**Step 3️⃣ - Format the Release content**

The content of the release will include information coming from the different pull request, issues,
and commits. This information can be included automatically into the release notes using different
patterns. These patterns are described in the `release-drafter.yml` file inside `.github` folder:

The following example is a full example using different categories of information to add into the
release notes.

```yaml
# This release drafter follows the conventions from https://keepachangelog.com

name-template: 'v$RESOLVED_VERSION'
tag-template: 'v$RESOLVED_VERSION'
template: |
  ## What Changed 👀
  
  $CHANGES

  **Full Changelog**: https://github.com/$OWNER/$REPOSITORY/compare/$PREVIOUS_TAG...v$RESOLVED_VERSION
categories:
  - title: 🚀 Features
    labels:
      - feature
      - enhancement
  - title: 🐛 Bug Fixes
    labels:
      - fix
      - bug
  - title: ⚠️ Changes
    labels:
      - changed
  - title: ⛔️ Deprecated
    labels:
      - deprecated
  - title: 🗑 Removed
    labels:
      - removed
  - title: 🔐 Security
    labels:
      - security
  - title: 📄 Documentation
    labels:
      - docs
      - documentation      
  - title: 🧩 Dependency Updates
    labels:
      - deps
      - dependencies
    collapse-after: 5

change-template: '* $TITLE (#$NUMBER)'
change-title-escapes: '\<*_&' # You can add # and @ to disable mentions, and add ` to disable code blocks.
  
exclude-labels:
  - skip-changelog
```

**Step 4️⃣ - Update Changelog file**

After a new version is released, we want to update the changelog with the latest changes, as we
are doing with the release notes. We can automate it using another amazing
GitHub Action - [Changelog Updater](https://github.com/marketplace/actions/changelog-updater).

This action can be integrated into another workflow (i.e: `update-changelog.yml` inside of
the `.github/workflows` folder). This workflow can be similar to:

```yaml
name: 📄 Update Changelog 📄

on:
  release:
    types: [released]

jobs:
  update:
    name: Update Changelog
    runs-on: ubuntu-latest
    permissions:
      # Give the default GITHUB_TOKEN write permission to commit and push the 
      # updated CHANGELOG back to the repository.
      # https://github.blog/changelog/2023-02-02-github-actions-updating-the-default-github_token-permissions-to-read-only/
      contents: write    

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Update Changelog
        uses: stefanzweifel/changelog-updater-action@v1
        with:
          latest-version: ${{ github.event.release.tag_name }}
          release-notes: ${{ github.event.release.body }}

      - name: Commit updated Changelog
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          branch: main
          commit_message: '🔖 Update changelog'
          file_pattern: CHANGELOG.md
```

So, this workflow will start when a new release is released (`types: [released]`), including
the changes from previous release and committing the change into the `main` branch of our repo.

**Step 5️⃣ - Linking release and update changelog workflows**

There is an issue reported [here](https://github.com/stefanzweifel/changelog-updater-action/discussions/30)
about how to automatically trigger the update changelog workflow from the release workflow. The workaround
to fix it requires adding a new secret (i.e: `PERSONAL_ACCESS_TOKEN`) into your repo:

{:refdef: style="text-align: center;"}
[![](/images/2023/06/github/gh-secrets.avif "GitHub Repo secrets")]({{site.url}}/images/2023/06/github/gh-secrets.avif)
{: refdef}

**Step 6️⃣ - Release a new version**

Now, it is very simple, just follow your development workflow, using your pull-request life cycle, add the labels
of your own repository, and then tag a new version when you are ready to do it.

Push it into your repo:

```shell
git tag v1.2.1 -m "Version 1.2.1"
git push origin v1.2.1
```

The workflows run as expected:

{:refdef: style="text-align: center;"}
[![](/images/2023/06/github/gh-actions.avif "Workflows executed")]({{site.url}}/images/2023/06/github/gh-actions.avif)
{: refdef}

... a new release is created, including the notes:

{:refdef: style="text-align: center;"}
[![](/images/2023/06/github/gh-new-release.avif "New GitHub Release")]({{site.url}}/images/2023/06/github/gh-new-release.avif)
{: refdef}

... and the changelog is updated too:

{:refdef: style="text-align: center;"}
[![](/images/2023/06/github/gh-changelog.avif "Changelog updated")]({{site.url}}/images/2023/06/github/gh-changelog.avif)
{: refdef}

This is ...

{:refdef: style="text-align: center;"}
[![](/images/bitmoji/super-awesome.avif "Super Awesome")]({{site.url}}/images/bitmoji/super-awesome.avif)
{: refdef}

🤖🚩 Happy automating releasing!!! 🤖🚩

## References

This blog post is my own summary about this process, but it was based from the content
and experience of others, such as:

* [Stop writing your changelogs manually](https://tiagomichaelsousa.dev/articles/stop-writing-your-changelogs-manually)
* [Release Drafter GitHub Action](https://github.com/marketplace/actions/release-drafter)
* [Changelog Update GitHub Action](https://github.com/marketplace/actions/changelog-updater)

My kudos ❤️ to all of them!!!
