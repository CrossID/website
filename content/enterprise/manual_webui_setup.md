---
title: Manual Web UI Setup
linktitle: Manual Web UI Setup
description: How to manually serve Web UI
date: 2018-11-04
publishdate: 2018-11-04
lastmod: 2018-11-04
categories: [enterprise, onprem, private-cloud, setup, installation, manual]
layout: single
menu:
  docs:
    parent: "enterprise"
    weight: 20
weight: 20
sections_weight: 20
draft: false
toc: true
---

This page explains how to serve web ui static resources,

It is intended for those who can't or don't want to use an automated approach such docker or Ansible book.

# Pre requisites

- Linux x64 (such RHEL 7.x) with root / sudo access, see [backend hardware requirements](hardware-requirements) for more info.
- Running backend.
- Setup config files.

# Setup layout

Our folder layout will look like:

```
pkg - installation packages
serve - static files to be served
```


# Manually copy installation files

Manually copy frontend files to $HOME/pkg:

- `crossid-webui-vx.x.x.tgz`
- `serve-linux-amd64`
- `crossid-webui.service`


# Extract tarball and static web server

Setup temporary env vars (as root):

- Replace _v1.0.0_ with the relevant binary version.

```bash
export CID_INST_WEBUI_VER=v1.0.0
useradd crossid-webui
passwd crossid-webui
su - crossid-webui
mkdir -p $HOME/bin $HOME/pkg -p $HOME/webui-${CID_INST_WEBUI_VER}
ln -s $HOME/webui-${CID_INST_WEBUI_VER} $HOME/serve
mkdir -p $HOME/serve/common
```

Extract the web ui and the static web server:

```bash
tar zxvfl $HOME/pkg/crossid-webui-${CID_INST_WEBUI_VER}.tgz -C $HOME/serve
cp $HOME/pkg/serve-linux-amd64 $HOME/bin
chmod +x $HOME/bin/serve-linux-amd64
```


# Running as a systemd service

```bash
sudo cp $HOME/pkg/crossid-webui.service /etc/systemd/system/crossid-webui_9005.service
sudo systemctl daemon-reload
sudo systemctl start crossid-webui_9005
# should show active (running)
sudo systemctl status crossid-webui_9005
```

Try to hit frontend by:

curl http://localhost:9005/admin
