---
title: Manual Traefik installation
linktitle: Manual Traefik installation
description: How to manually install Traefik Proxy
date: 2018-11-23
publishdate: 2018-11-23
lastmod: 2018-11-23
categories: [enterprise, onprem, private-cloud, setup, installation, manual, proxy]
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

This page explains how to install [Traefik](https://traefik.io) proxy server

It is intended for those who can't or don't want to use an automated approach such docker.


```bash
useradd crossid-proxy
passwd crossid-proxy
su - crossid-proxy
mkdir -p $HOME/bin $HOME/etc $HOME/pkg
```

Manually copy the following files to _$HOME/pkg_

- `traefik_linux-amd64`
- `traefik.conf`
- `traefik.service`
- `csr_details.txt`

Put files in the right locations

```bash
cp $HOME/pkg/traefik_linux-amd64 $HOME/bin
chmod +x $HOME/bin/traefik_linux-amd64
cp $HOME/pkg/traefik.conf $HOME/etc
```

Replace _serverName_ with the name of the server, e.g.,:

```bash
sed -i s/serverName/`hostname`/g $HOME/etc/traefik.conf
```


### SSL

Create a CSR:

```bash
cp $HOME/pkg/csr_details.txt
```

Manually modify the _csr-details.txt_ file, please note that _CN_ and _alt_names_ section must match the dns to be used by clients.

Issue a CSR:

```bash
openssl req -new -sha256 -nodes -out crossid.csr -newkey rsa:2048 -keyout crossid.key -config <( cat csr_details.txt )
```

Tip: you can temporarily generate a self signed certificate until the signed certificate is returned by:

```bash
openssl x509 -signkey $HOME/etc/crossid.key -in $HOME/etc/crossid.csr -req -days 365 -out $HOME/etc/crossid.pem
```


### Check that proxy is working


```bash
$HOME/bin/traefik_linux-amd64 --configFile=$HOME/etc/traefik.conf
```


At this point UI should be available in URL such: _https://serverNAme:8443/admin_


### Running as a systemd service

```bash
sudo cp $HOME/pkg/traefik.service /etc/systemd/system/crossid-proxy.service
sudo systemctl daemon-reload
sudo systemctl start crossid-proxy
# should show active (running)
sudo systemctl status crossid-proxy
```


### Redirect 8443/8080 to standard ports

```bash
sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8443
```
