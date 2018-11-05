---
title: Manual Backend Setup
linktitle: Manual Backend Setup
description: How to manually setup and configure backend
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

This page explains how to setup the backend manually,

It is intended for those who can't or don't want to use an automated approach such docker or Ansible book.

# Pre requisites

- Linux x64 (such RHEL 7.x) with root / sudo access, see [backend hardware requirements](hardware-requirements) for more info.
- Requires Mongo service up and running, optionally configured with auth and SSL.
- Crossid backend binary.
- Setup config files.

Tip: It is recommended to set up a machine with tools such _Notepad++, Winscp, Postman, Robo3t_ to please the procedure.


# Setup layout

Our folder layout will look like:

```
pkg - installation packages
bin - binary and shell scripts
etc - configuration files
data - data import files
```

# Preparations

Add `CID_HOME` env var to point the root application location, can refer to $HOME directory or to folders such `/opt/crossid`

```
echo export CID_HOME=$HOME >> ~/.bash_profile
source ~/.bash_profile
```

Copy all installation packages into `$CID_HOME/pkgs`

# Manually copy installation files


Manually copy backend files to $HOME/pkg:

- `crossid-vx.x.x-linux-amd64.tar.gz`
- `crossid.env`
- `crossid.service`
- `cid.sh` (script)
- `cid_start.sh` (script)
- `cid_import.sh` (script)
- `cid_data_import` (folder)


# Installation

Setup temporary env vars (as root):

- Replace _indexia_ with the relevant tenant id.
- Replace _v1.0.0_ with the relevant binary version.

```bash
export CID_INST_BACKEND_VER=v1.0.0
useradd crossid
passwd crossid
su - crossid
mkdir -p $HOME/bin $HOME/etc $HOME/log $HOME/data $HOME/pkg
```


# Extract backend binary, configs and scripts

```bash
tar zxvfl $HOME/pkg/crossid-${CID_INST_BACKEND_VER}-linux-amd64.tar.gz -C $HOME/bin
# note: another option is to use alternative instead
ln -s $HOME/bin/crossid-${CID_INST_BACKEND_VER}-linux-amd64 $HOME/bin/crossid
cp $HOME/pkg/crossid.env $HOME/etc
cp $HOME/pkg/*.sh $HOME/bin
chmod +x $HOME/bin/*.sh
```

Edit _etc/crossid.env_ and change _CID_DB_CONNECTION_STRING_, _CID_JOBSCHED_TENANT_ID_ and _CID_SERVER_.

Start the server by: `cid.sh server start`, should prompt with _Server is ready to accept connections._


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


# Troubleshooting

## certificate signed by unknown authority" when starting server

If error such as below is displayed:

```
FATA[0000] Failed to create connection                   reason="x509: certificate signed by unknown authority"
```

Then it is required to add the self signed certificate used by Mongo as a root CA to the Linux system trusted store:

```bash
sudo cp mongod.pem /etc/pki/ca-trust/source/anchors
sudo update-ca-trust
```

Then to start server again.

# Extra

## Creating a tenant

A tenant can be created using the _crossid_ binary by running:

```bash
cid.sh tenant create --file $HOME/data/tenant.json --user-file $HOME/data/import/resources/cid_user_system.json
```

Where _tenant.json_ is the tenant doc and _cid_user_system.json_ is an initial super user to import into the tenant.