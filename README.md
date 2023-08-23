# rmarting.github.io site

![License](https://img.shields.io/github/license/rmarting/rmarting.github.io?style=plastic)
![Main Lang](https://img.shields.io/github/languages/top/rmarting/rmarting.github.io)
![Languages](https://img.shields.io/github/languages/count/rmarting/rmarting.github.io)
![Last Commit](https://img.shields.io/github/last-commit/rmarting/rmarting.github.io)
[![gitmoji](https://img.shields.io/badge/gitmoji-%20😜%20😍-FFDD67.svg?style=flat-square)](https://gitmoji.dev)

This is the repo of [my personal blog](https://blog.jromanmartin.io), enjoy it better on live.

If you want to contribute, please, review my how you could do it [here](./CONTRIBUTING.md), and
don't forget to follow the [Code of Conduct](./CODE_OF_CONDUCT.md).

This blog is a customization of the [Flexible-Jekyll Theme](https://github.com/artemsheludko/flexible-jekyll).

## Building locally

This site is built with [Jekyll](https://jekyllrb.com). If you want to build and run
locally then you need to follow [the instructions](https://jekyllrb.com/docs/installation/) for
your local environment.

**NOTE**: Github-Pages uses `Jekyll 3.9`, which isn’t compatible with Ruby 3. Keep in mind for your
local environment.

**TL;DR**: This is the sort way to install Ruby 2.7.2 in a Fedora 38 workstation to build and run
locally this site. For Fedora workstations, [this reference](https://developer.fedoraproject.org/tech/languages/ruby/ruby-installation.html)
using `rbenv`, or [this another](https://tecadmin.net/install-ruby-on-fedora/) using `rvm` can help you.
For other environments, please contribute with here with your knowledge and experience.

Main steps

```shell
on 🎩 ❯ rbenv install 2.7.2
on 🎩 ❯ rbenv global 2.7.2
on 🎩 ❯ ruby -v
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-linux]
on 🎩 ❯ gem install jekyll bundler
on 🎩 ❯ bundle install
```

If you have already installed the gems, use `bundle update` for updating.

## Running locally

To run locally, to test and verify this blog site:

```shell
bundle exec jekyll serve
```

Your local site is available at [http://127.0.0.1:4000/](http://127.0.0.1:4000/).
