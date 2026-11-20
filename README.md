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

This site is built with [Jekyll](https://jekyllrb.com). Local preview uses **Jekyll 4.4** via Bundler.

### Prerequisites

Install Ruby and the build tools needed for native gems:

```shell
sudo dnf install ruby ruby-devel rubygems rubygem-bundler \
  gcc gcc-c++ make openssl-devel libyaml-devel \
  readline-devel zlib-devel libffi-devel
```

Confirm you are on a supported Ruby for this `Gemfile` (3.4+ / 4.x):

```shell
ruby -v
```

Official Jekyll install notes: [jekyllrb.com/docs/installation](https://jekyllrb.com/docs/installation/).
Fedora Ruby overview: [developer.fedoraproject.org — Ruby](https://developer.fedoraproject.org/tech/languages/ruby/ruby-installation.html).

### Install gems

From the repository root:

```shell
bundle install
```

To refresh gems later:

```shell
bundle update
```

## Running locally

```shell
bundle exec jekyll serve
```

Your local site is available at [http://127.0.0.1:4000/](http://127.0.0.1:4000/).
