---
layout:     post
title:      ":wrench: How to Install Jekyll in Fedora 32"
date:       2020-09-25 09:28:53 +0200
toc:        true
comments:   true
img:        fedora-logo.png
tags: 
- How-to 
- fedora32
- ruby 
- GitHub
---

[Jekyll](https://jekyllrb.com/) is the engine behind [GitHub Pages](https://pages.github.com/)
to build, deploy and publish your Blog Site easily. Using Jekyll, you can blog using
beautiful Markdown syntax, and without having to deal with any databases.

[Roman's Blog](https://rmarting.github.io) is published using this amazing technology, however
my first steps were not easy. 

To use it basically means to execute the following commands:

{% highlight bash %}
❯ gem install bundler jekyll
❯ jekyll new rmarting.github.io
❯ cd rmarting.github.io
❯ bundle exec jekyll serve
   Server address: http://127.0.0.1:4000/
{% endhighlight %}

However the reality was not so easy. This blog post described the steps done to install and use
on my Fedora 32.

## Installation errors

This simple command was a nightmare:

{% highlight bash %}
>  gem install bundler jekyll
{% endhighlight %}

Because I found so many installation errors such as:

{% highlight bash %}
Building native extensions. This could take a while...
ERROR:  Error installing jekyll:
        ERROR: Failed to build gem native extension.

    current directory: /home/rmarting/.gem/ruby/gems/eventmachine-1.2.7/ext
/usr/bin/ruby -I /usr/share/rubygems -r ./siteconf20200925-983033-i0nt3z.rb extconf.rb
mkmf.rb can't find header files for ruby at /usr/share/include/ruby.h

You might have to install separate package for the ruby development
environment, ruby-dev or ruby-devel for example.

extconf failed, exit code 1

Gem files will remain installed in /home/rmarting/.gem/ruby/gems/eventmachine-1.2.7 for inspection.
Results logged to /home/rmarting/.gem/ruby/extensions/x86_64-linux/2.7.0/eventmachine-1.2.7/gem_make.out
1 gem installed
{% endhighlight %}

And:

{% highlight bash %}
Building native extensions. This could take a while...
ERROR:  Error installing jekyll:
        ERROR: Failed to build gem native extension.

    current directory: /home/rmarting/.gem/ruby/gems/eventmachine-1.2.7/ext
/usr/bin/ruby -I /usr/share/rubygems -r ./siteconf20200925-983599-if4elb.rb extconf.rb
checking for -lcrypto... *** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.

# Reduced log messages to avoid be a large blog

To see why this extension failed to compile, please check the mkmf.log which can be found here:

  /home/rmarting/.gem/ruby/extensions/x86_64-linux/2.7.0/eventmachine-1.2.7/mkmf.log

extconf failed, exit code 1

Gem files will remain installed in /home/rmarting/.gem/ruby/gems/eventmachine-1.2.7 for inspection.
Results logged to /home/rmarting/.gem/ruby/extensions/x86_64-linux/2.7.0/eventmachine-1.2.7/gem_make.out
1 gem installed
{% endhighlight %}

And this other one:

{% highlight bash %}
Building native extensions. This could take a while...
ERROR:  Error installing jekyll:
        ERROR: Failed to build gem native extension.

    current directory: /home/rmarting/.gem/ruby/gems/eventmachine-1.2.7/ext
/usr/bin/ruby -I /usr/share/rubygems -r ./siteconf20200925-984930-xt8hum.rb extconf.rb

# Reduced log messages to avoid be a large blog

make: g++: Command not found
make: *** [Makefile:237: binder.o] Error 127

make failed, exit code 2

Gem files will remain installed in /home/rmarting/.gem/ruby/gems/eventmachine-1.2.7 for inspection.
Results logged to /home/rmarting/.gem/ruby/extensions/x86_64-linux/2.7.0/eventmachine-1.2.7/gem_make.out
1 gem installed
{% endhighlight %}

## Prepare your Fedora 32

After some researching (ok, googling it) I found that you need to install something else that **ruby**. The
final list of dependencies you have to install is:

{% highlight bash %}
❯ sudo dnf install rubygems ruby-devel gcc rpm-build gcc-c++ make zlib-devel
{% endhighlight %}

Then you could come back to the original commands to execute locally your GitHub pages:

{% highlight bash %}
❯ gem install bundler jekyll
❯ bundle exec jekyll serve
   Server address: http://127.0.0.1:4000/
{% endhighlight %}

Enjoy it!

## References

* [Jekyll](https://jekyllrb.com/)
* [Setting up a GitHub Pages with Jekyll](https://docs.github.com/en/github/working-with-github-pages/setting-up-a-github-pages-site-with-jekyll)
