---
layout:      post
title:       "📛 Improving a GitHub Repo (II)!"
subtitle:    How to use a GitHub template repository to setting up your repository fast and easily.
description: How to use a GitHub template repository to setting up your repository fast and easily.
date:        2023-06-20 09:00:00 +0200
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

My first post of [📛 Improving a GitHub Repo](https://blog.jromanmartin.io/2023/06/12/Improving-a-gh-repository.html) describes
many good things to add in any GitHub repository to be more productive and professional. However, that stuff can be hard
to do every time a new repository is created, and we can forget to add something great. I found a way to accelerate this process
and also do not forget to add anything great: [GitHub Repository templates](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository).

GitHub template repository is the best way to replicate an standard structure, including folders, documentation, workflows, branches, and
any file required to set up a new project. Using this pattern we can homogenize the structure of any repository of your organization,
or also your own projects, easily and saving a lot of time. If you need to standardize your projects, or you need to create many projects
on demand, definitely a template repository is your tool.

As summary, the most great benefits of using repository template I found are:

* ⌛ Spend less time repeating code
* 🌟 Focus on building new things
* 🦾 Less manual configuration
* 📝 Sharing boilerplate code across the code base

And, the main features of a repository template are:

* Copy the entire repository files to a brand new repository
* Every template has a new url endpoint called `/generate`
* Share repository template through your organization or other GitHub users

## My own repository template

I create mw own GitHub Template repository in here: [https://github.com/rmarting/gh-repo-template](https://github.com/rmarting/gh-repo-template) including
all the things described in my previous [post](https://blog.jromanmartin.io/2023/06/12/Improving-a-gh-repository.html), or new things added along the time.

My template repository includes things such as:

* Initial content files aligned with the common patterns in any Open Source project: Contribution guide, Code of Conduct, contributors, ...
* GitHub templates to report issues or open Pull Requests.
* Standard badges to summarize the repository.
* Standard workflows to release versions, or to implement Continuous Integration pipelines

So, creating a new repository and setting up it takes few seconds and steps. Amazing!!!

Do you have ideas, or comments about how to improve a template repository? Looking forward to hearing you with more contributions into
my template repo, or adding comments in this post. 

🤖🚩 Happy creation of new projects!!! 🤖🚩

{:refdef: style="text-align: center;"}
[![](/images/bitmoji/happy-coding.avif "Happy coding!!!")]({{site.url}}/images/bitmoji/happy-coding.avif)
{: refdef}
