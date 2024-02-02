---
title: Deploying a static site on Cloudflare Pages with Hugo and Terraform
description: Using the Hugo framework to build assets for a static website hosted on a Terraform-managed Cloudflare Pages project
slug: hugo-cloudflare-terraform
date: 2024-02-01 00:00:00+0000
categories:
    - Terraform
tags:
    - terraform
    - hugo
    - cloudflare
---

This tutorial will detail the process of creating and hosting a static website on [Cloudflare Pages](https://pages.cloudflare.com/) with [Hugo](https://gohugo.io) and [Terraform](https://terraform.io). The described process is responsible for the generation of this website, the source code of which is contained in this [repo hosted on GitHub](https://github.com/selfsealingstembolts/selfsealingstembolts.com).
  
## Overview

We'll use the following components to create our static site, in order of appearance:

- [**Hugo**](https://gethugo.io) - A framework for generating static sites using a combination of Go templates and markdown

- [**Terraform**](https://terraform.io) - Use infrastructure-as-code to create a project within [Cloudflare Pages](https://pages.cloudflare.com), configuring the website source and build environemnt.
  - Optionally, set up [Cloudflare Zero Trust](https://www.cloudflare.com/zero-trust/) to limit access to the development and/or production versions of our website deployment.

### Advantages

- **Simplicity** - Creating content is easy; content can be written in markdown, and the build and deploy process is handled automatically on every commit.
- **Performance** - Static websites are incredibly performant, a metric crucial to both a positive user experience as well as search engine ranking.
- **Cost** - Hosting on Cloudflare Pages is [free for static content](https://developers.cloudflare.com/pages/platform/limits/).

### Assumptions

This tutorial assumes you have access to and a familiarity with the following:

- **Cloudflare Account** - A free account at [Cloudflare](https://cloudflare.com).

- **Domain** - A registered domain name; this tutorial will assume the domain is registered with Cloudflare, but this is not mandatory.

- **Linux/macOS** - Access to a Linux or macOS system, as well as a basic understanding of interacting with a terminal.

- **Git** - An account with a Git hosting provider (e.g. [GitHub](https://github.com), [GitLab](https://gitlab.com), [BitBucket](https://bitbucket.org), or a publicly accessible self-hosted solution), as well as a basic understanding of interacting with it via command line. 

  
## Creating a website with Hugo

### Prerequisites

Hugo recommends the following for an extended featureset:

- [Git](https://git-scm.com/)
- [Go](https://go.dev/)

The installation of these prerequisites is outside the scope of this tutorial, however **they will be required** if following this tutorial as written.

### Installation

The [documentation](https://gohugo.io/installation/) provided by the project itself is comprehensive, but we'll provide an abridged version here. Generally speaking, Hugo will be installed with your package manager. 

A few examples of some common Linux distributions, as well as macOS, are as follows:

**Debian/Ubuntu**

```bash
sudo apt install hugo
```

**Fedora/RedHat/CentOS**

```bash
sudo dnf install hugo
```

**Arch Linux**

```bash
sudo pacman -S hugo
```

**macOS (Homebrew)**

```bash
brew install hugo
```

### Initialization

Once installed, it's time to initialize a new project. Create a new directory for the project - we'll use `~/Development/www/selfsealingstembolts.com` here. This directory will contain two directories: `terraform` and `hugo`, each containing the resources of its respective component.

    
```bash
mkdir -p ~/Development/www/selfsealingstembolts\.com/{terraform,hugo}
```

Move to the newly created project directory, and initialize a new site with Hugo.

```bash
cd ~/Development/www/selfsealingstembolts\.com
hugo new site ./hugo
```

### Theming

Next up, let's select a theme. This tutorial will use [Stacks](https://stack.jimmycai.com/), but it is by no means the only (or situationally appropriate) option. 
> The Hugo project conveniently provides a [list of themes](https://themes.gohugo.io/) for your perusal. However, selecting a theme other than the one specified by this tutorial will alter some of the configuration elements and site structure described here.  

Themes can be installed either as a Git submodule or a Hugo module. Installation as a Hugo module is the most canonical method, therefore it is the method we will use here. First, we must convert our project to a Hugo module.

```bash
cd hugo
hugo mod init github.com/<user>/<repo>
```

where `<user>` is your GitHub username and `<repo>` is the name of your git repository.


