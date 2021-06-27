---
layout: post
title: Install PIP on Synology NAS
tags: [python, synology, howto]
cover-img: "/assets/img/head/console.jpg"
---

Python3 can be installed via the Synology DSM Installation Center, but pip is not installed. Here is a short how to on how to install pip.

## Install pip

```cli
sudo python3 -m ensurepip
```

## Update pip

```cli
sudo python3 -m pip install --upgrade pip
```

## Check pip version

```cli
python3 -m pip -V
```

## Install a package

```cli
sudo python3 -m pip install XYZ
```
