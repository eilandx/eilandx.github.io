---
title: Setting up local testing for Jekyll with the Chirpy theme
date: 2024-01-31 
categories: [Guide, Tutorial]
tags: [jekyll, chirpy, tutorials]     # TAG names should always be lowercase
author: CrimsonCloak
---

# Setting up local testing for Jekyll

## Goal

In order to test our Jekyll setup without having to endlessly build and deploy on GitHub actions, we need a local serving of Jekyll. This short guide walks you throught installing the required software in order to test locally before you push remotely on your GitHub repository to trigger your GitHub Action.

## Steps

### Install Ruby

Make sure the latest version of [Ruby](https://www.ruby-lang.org/en/documentation/installation/) is installed on your system.

### Install Bundler

With most modern Ruby distributions, Bundler comes pre-installed. If this is not the case, be sure to [download Bundler manually](https://bundler.io/).

### Install Ruby Gems

Using Bundle, we can then install the required Ruby Gems (~packages) by running the following command in the repository:

```console
bundle install
```

### Serve Jekyll server

After installing the required Ruby Gems, we can then serve the Jekyll server locally on our device. The following command can be used:

```console
bundle exec jekyll serve
```


