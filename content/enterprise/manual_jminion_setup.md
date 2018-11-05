---
title: Manual Java Minion Setup
linktitle: Manual Java Minion Setup
description: How to manually setup and configure java minion
date: 2018-11-05
publishdate: 2018-11-05
lastmod: 2018-11-05
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

This page explains how to setup the minion manually,

# Pre requisites

- Linux x64 (such RHEL 7.x) with root / sudo access.
- Running backend.


# Setup layout

Our folder layout will look like:

```
pkg - installation files
```


# Manually copy installation files

Manually copy files to $HOME/pkg:

- `minion-x.x.x-dist.tar.gz`
- `crossid-minion.service`


# Extract tarball

```bash
useradd crossid-minion
passwd crossid-minoon
su - crossid-minion
# Replace _v1.0.0_ with the relevant binary version.
export CID_INST_JMINION_VER=1.0.0
mkdir -p $HOME/pkg $HOME/minion-${CID_INST_JMINION_VER}
ln -s $HOME/minion-${CID_INST_JMINION_VER} $HOME/minion
```

Extract the minion tarball:

```bash
tar zxvfl $HOME/pkg/minion-${CID_INST_JMINION_VER}.tar.gz -C $HOME/minion
```

# Configuration

The configuration resides within `~/minion/etc/minion.properties` file where keys are:

- network.url = the binding url in the form of _http://host_
- network.port - the binding port
- security.jwt.secret - the secret where JWT tokens are signed with by the backend


# Running as a systemd service

```bash
sudo cp $HOME/pkg/crossid.service /etc/systemd/system/crossid_9000.service
sudo systemctl daemon-reload
sudo systemctl start crossid_9000
# should show active (running)
sudo systemctl status crossid_9000
```

Try to hit the backend API:

_http://serverName:9000/api/v2/apps/cid_, you should get auth failure message.
