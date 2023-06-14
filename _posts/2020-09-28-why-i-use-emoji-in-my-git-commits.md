---
layout:     post
title:      ":bulb: Why I use emojis in my Git commits"
date:       2020-09-28 08:00
toc:        true
comments:   true
img:        git-merge.avif
fig-caption: Git
fig-copy:    true
fig-author:       Markus Spiske
fig-author-link:  https://www.pexels.com/@markusspiske
fig-gallery:      Pexels
fig-gallery-link: https://www.pexels.com/
tags: 
- How-to
- git
- emoji
- development
- productivity
---

We use [Emojis](https://getemoji.com/) every day in different channels like Telegram, Slack, WhatsApp,
Google Chats, ... They are a fast way to communicate in this visual world. So I use them too in my 
git commits in the same way.

Around one year ago I started to use them after collaborate in a repo where some colleagues used emojis
to give a visual summary of the commits. In my homuilde opinion writing git commit messages 
is an [art](https://chris.beams.io/posts/git-commit/) that I am not doing well in many cases. Emojis
help me to reduce the commit message and also add a visual message for others.

The UI of the most common and extended Git SCM integrates the emojis when you are browsing the repo, and
the experience as a user is more clear, visual, fun and easy to follow. I used in different SCM such as:

* [GitHub](https://github.com/): Full integrated :star2:.
* [GitLab](https://gitlab.com/): Very well integrated :star:.
* [Bitbucket](https://bitbucket.org/): Very bad integration, but my commits have emojis :grin:.
* [Gitea](https://gitea.io/): Enough integration (but I did few commits) :yum:.
* [Gogs](https://gogs.io/): Enough integration (but I did few commits) :yum:.

Your repo could have a look and feel similar to: 

[![](/images/2020/09/gh-emoji/git-repository-layout.avif "Git Repository")]({{site.url}}/images/2020/09/gh-emoji/git-repository-layout.avif)

Nice, right?

So after this time using them, I could conclude that the main reasons to continue is:

* Be Cool? Hipster? Not at all
* Simple visualization of the status of the repo
* Funny documentation and easy to follow (not only in commits)
* Simple and direct messages
* Can I do it? Go ahead

## Top List of emojis

The full list of emojis is so large, but after some time I have a top list of the
most common emojis in my daily development. 

I could summarize my top list of emojis in:

* :tada: ```:tada:```: Everything starts with the first commit.
* :memo: ```:memo:```: Documentation is important in any project.
* :sparkles: ```:sparkles:```: Adding new features in your project.
* :bug: ```:bug:```: Fixing your code ... not always is working fine.
* :fire: ```:fire:```: Removing things.
* :bookmark: ```:bookmark:```: Releasing a version
* :arrow_down: :arrow_up: ```:arrow_down:``` ```:arrow_up:```: Managing your dependencies versions.
* :recycle: ```:recycle:```: Refactoring code.
* :wrench: ```:wrench:```: Configuring the project.
* :twisted_rightwards_arrows: ```:twisted_rightwards_arrows:```: Merging branches.
* :truck: ```:truck:```: Moving or renaming resources.
* :bulb: ```:bulb:```: Comments in the source code are always needed
* :see_no_evil: ```:see_no_evil:```: You don't need to track everything (.gitignore is here)
* :wastebasket: ```:wastebasket:```: Deprecating or cleaning code

Choose your own ones.

## Tools

I know that it is very hard to learn all the emojis available, however there are some useful tools
to help you. 

From my personal experience, I :cupid: [gitmoji-cli](https://www.npmjs.com/package/gitmojis) by
Carlos Cuesta. This is a simple, and easy CLI to add emojis in your commits. This tool
includes helpers, searches to choose the right emoji before your commit or when your are committing:

{:refdef: style="text-align: center;"}
[![](/images/2020/09/gh-emoji/gitmoji-commit.avif "gitmoji commit")]({{site.url}}/images/2020/09/gh-emoji/gitmoji-commit.avif)
{: refdef}

The [git repo](https://github.com/carloscuesta/gitmoji) has everything you need to start to use it. 

Happy committing !!! :beers::construction_worker:
