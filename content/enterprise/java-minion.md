---
title: Java Minion
linktitle: Java Minion
description: The Java Minion intends to communicate with enterprise protocols such JDBC, WS, etc.
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
categories: [enterprise, onprem, private-cloud]
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

Java wins them all when it comes to enterprise protocols, it is possible to implement applications logics for applications that are some kind of RDBMS (e.g., "Oracle", "MSSQL"),
Web Services or apps that have Java SDK.


## Installation

### Docker Installation

Create and run a container of the jminion image: `docker run --name cid-jminion -p9000:9000 crossid/jminion`

### Manual installation

Please download and install Java SE 8

- [Java JRE 8u151+](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)

Or in ubuntu:

```bash
sudo apt-get update
sudo apt-get install default-jre
java -version
# openjdk version "1.8.0_151"
```


Extract the tar into some folder by and start the service:

```bash
tar zxvfl _minion-x.x.x-dist-tar.gz_ -C /some/folder
cd /some/folder
./bin/start.sh
# INFO: Started listener bound to [localhost:9000]
```

## Minion Configuration

Configuration can be found in `etc/minion.properties` file.

### Authentication

Minion authenticate requests using a _JWT bearer token_, the secret of the token is defined in the _security.jwt.secret_ property, please set this property with some complicated random string (such: `openssl rand -base64 12`)

Minion restart is required for the config changes to take affect.

#### Obtaining a token from backend

You can use the backend CLI in order to create a token for the minion, in _crossid-standalone_ folder run:

```bash
./bin/cid.sh token generate --sub-type minion --tenant-id dev --exp 900000 --secret <secret>
```

Replace _<secret>_ with the secret defined in _security.jwt.secret_


Lets check that the token works:

curl http://localhost:9000/api/scim/v2/app1/Users -H 'Authorization: Bearer <token>'

Replace _<token>_ with the token you got.

It should not get authorization error rather some error that app1 doesn't exist such:

{{< highlight json >}}
{
  "schemas": [
    "urn:ietf:params:scim:api:messages:2.0:Error"
  ],
  "status": "404",
  "scimType": "invalidValue",
  "detail": "Could not load app manifest [dev:app1]: ..."
}
{{< / highlight >}}

## Deploying application in minion

This example demonstrate how to deploy an app logic for MariaDB application.

### Download relevant drivers

MariaDB requires a JDBC driver that can be downloaded from [here](https://mariadb.com/downloads/connector), _jar_ drivers should be put under the _minion/lib_ folder.

### Put the app logic class in the lib folder

Putting a single class:

If the package of the app logic class is _io/crossid/customers/indexia_ the appLogic class should be put under _minion/lib/io/crossid/customers/indexia_.

Putting a jar:

If app logic is archived in a jar, just put the jar in the _minion/lib_ folder

### Put the app manifest in the apps folder

Put the _app manifest_ json file in _minion/apps/<tenant>_

For example if the tenant name is _dev_, the app manifest should be put under _minion/apps/dev_