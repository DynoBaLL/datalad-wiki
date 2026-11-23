---
myst:
    html_meta: {
        "description lang=en": "A wiki about using DataLad in UNIGE infrastructure.",
        "keywords": "DataLad, UNIGE, wiki",
        "property=og:locale": "en_US"
    }
---

```{include} ../links.md
```

# Guide

This guide explains how to install {term}`datalad`, set up a project on the University of Geneva's infrastructure, and use it day to day.

## 1. Install

::::{grid} 1
:gutter: 3

:::{grid-item-card} {fas}`download` Install
:link: install
:link-type: doc

Install uv, datalad, SSH keys and the GitLab tools needed by all setups.
:::
::::

## 2. Setup guides

All setups use {term}`gitlab` for version control and metadata. They differ in where the data is stored.

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card} Gitlab + NAS
:link: gitlab_nas
:link-type: doc
:img-top: ../_static/img/scenario_01.png
:img-alt: Gitlab + NAS setup

Data stored on the UNIGE {term}`NAS`, accessed through a {term}`VM`.
:::

:::{grid-item-card} Gitlab + S3
:link: gitlab_s3
:link-type: doc
:img-top: ../_static/img/scenario_02.png
:img-alt: Gitlab + S3 setup

Data stored on the UNIGE {term}`S3` object storage, accessed directly from every machine.
:::
::::

## 3. Usage

::::{grid} 1
:gutter: 3

:::{grid-item-card} {fas}`terminal` Usage
:link: usage
:link-type: doc

Everyday commands: clone, get, save and publish data.
:::
::::

```{toctree}
:hidden:
:caption: Install

install
```

```{toctree}
:hidden:
:caption: Setup guides

gitlab_nas
gitlab_s3
```

```{toctree}
:hidden:
:caption: Usage

usage
```
