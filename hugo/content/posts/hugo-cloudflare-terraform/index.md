---
title: Deploying a static site on Cloudflare Pages with Hugo and Terraform
description: Using the Hugo framework to build assets for a static website hosted on a Terraform-managed Cloudflare Pages project
date: 2024-02-01 00:00:00+0000
featured_image: "image.jpg"
categories: Terraform
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

Next up, let's select a theme. This tutorial will use [Diary](https://github.com/amazingrise/hugo-theme-diary), but it is by no means the only option available. 

> The Hugo project conveniently provides a [list of themes here](https://themes.gohugo.io/) for your perusal. However, selecting a theme other than the one specified by this tutorial will alter some of the configuration elements and site structure described here.  

Move to the `hugo` directory, where the theme will be installed as a Git submodule in `themes/diary` directory.

```bash
cd hugo
git submodule add https://github.com/AmazingRise/hugo-theme-diary.git themes/diary
```

We'll create a configuration file for the theme at `config.toml` with the following contents:

```toml
# config.toml
baseURL = "https://selfsealingstembolts.com/"
DefaultContentLanguage = "en" # Theme's display language, supports: en, fr, zh, zh-hant
languageCode = "en-us"
title = "Josh Weston"
copyright = "Copyright 2023-2024 Josh Weston"
theme = "diary"
# googleAnalytics = "UA-123-45"

[markup]
  [markup.highlight]
    codeFences = true
    guessSyntax = false
    hl_Lines = ""
    lineNoStart = 1
    lineNos = true
    lineNumbersInTable = true
    noClasses = true
    style = "gruvbox"
    tabWidth = 4

[params]
subtitle = "For the love of infrastructure, automation, and yamok sauce."
enableGitalk = false
enableGiscus = false

# Twitter Card and Open Graph settings
enableOpenGraph = true
enableTwitterCards = true
title = "Self-Sealing Stem Bolts" # will set 'og:site_name'
description = "A blog by Josh Weston"  # will set 'og:description'

[taxonomies]
   tag = "tags"
   category = "categories"

[[menu.main]]
url = "/categories"
name = "Categories"
weight = 2
[[menu.main]]
url = "/tags"
name = "Tags"
weight = 3
[[menu.main]]
url = "/posts"
name = "Archive"
weight = 1
[[menu.main]]
url = "/index.xml"
name = "RSS Feed"
weight = 4
```

> The most up-to-date version of this file can be found in this tutorial's companion [GitHub repo](https://github.com/selfsealingstembolts/selfsealingstembolts.com/blob/main/hugo/config.toml)

This configuration is not comprehensive; additional configuration options are detailed in the theme's [documentation](https://github.com/AmazingRise/hugo-theme-diary/wiki). It is, however sufficient to build a functioning site.
