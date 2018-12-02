---
title: Manual Mongo DB Setup
linktitle: Manual Mongo DB Setup
description: How to manually setup MongoDB
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

This page explains how to setup Mongo DB with authentication and SSL manually.

It is intended for those who can't or don't want to use an automated approach such docker or Ansible book.

# Pre requisites

- Linux x64 (such RHEL 7.x) with root / sudo access, see [backend hardware requirements](hardware-requirements) for more info.


# Setup layout

Our folder layout will look like:

```
pkg - installation files
data - where db files are stored
etc - configuration files
logs - log files
```

# Manually copy installation files

Manually copy into $HOME/pkg the following files:

- `mongodb-linux-x86_64-rhel70-4.x.x.tgz` - the installation tar ball
- `mongod.conf` - mongo daemon configuration file
- `mongod.service` - systemd service
- `mongo_as_crossid.sh` - shell file to login as crossid user
- `mongo_as_super.sh` - shell file to login as super user


# Basic Setup

Define the following temporary env vars:

- Replace _4.0.3_ with the relevant mongodb version.


```bash
export MONGODB_VER=4.0.3
```

Run as _root_ user:

```bash
useradd mongod
passwd mongod
su - mongod
```

As _mongod_ user:

```bash
# create folders layout
mkdir -p $HOME/pkg $HOME/data $HOME/etc $HOME/logs
```

Before continuing, 

As _mongod_ user:

```bash
# extract mongo archive
tar zxvfl $HOME/pkg/mongodb-linux-x86_64-rhel70-${MONGODB_VER}.tgz -C $HOME
cp $HOME/pkg/mongod.conf $HOME/etc
ln -s $HOME/mongodb-linux-x86_64-rhel70-${MONGODB_VER} $HOME/mongodb
```

Start the mongod daemon manually (blocks the terminal):

```bash
$HOME/mongodb/bin/mongod -f $HOME/etc/mongod.conf
```

# Enabling Replica Set

This is required in order to replicate data

Note: If SSL is going to be enabled, please set in _mongod.conf_ _net/bindIp_ to server name or _0.0.0.0_
This will make the primary replica to be defined with server name rather 127.0.0.1
Which causes SSL host name validation issues.

```bash
# invoke client anonymously as auth is not required yet
$HOME/mongodb/bin/mongo
rs.initiate()
# initial replica set can be verified using rs.conf()
rs.conf()
```
At this stage mongo shell prompt should indicate that replica name and replica member such: cid:PRIMARY>



# Enabling Auth

It is highly recommended to enable authentication as currently Mongo can be fully managed anonymously.

Note: The admin db is unique in MongoDB. Users with normal access to the admin database have read and write access to all **databases**.


To create a super user:

Note: the userAdminAnyDatabase role by itself doesn't allow the user to do anything else besides assigning arbitrary rights to arbitrary users.
to actually do something on the database, the user must have additional roles _readWriteAnyDatabase_, _dbAdminAnyDatabase_, _clusterAdmin_.
A user that has the roles above with the _userAdminAnyDatabase_ can do anything.


```bash
# invoke client anonymously as auth is not required yet
$HOME/mongodb/bin/mongo
use admin
db.createUser(
  {
    user: "super",
    pwd: "secret",
    roles: [ {role: "userAdminAnyDatabase", db: "admin"} ]
  }
)
```

Press ctrl-d to exit the client.

Enable the security section in _etc/mongod.conf_ and restart the mongod daemon.

At this point auth is enabled and no one can perform any action but the _super_ user.



Lets add a user for crossid backend, this user will be able to manage (read and write) the tenant db and the special _cid_ db.


```bash
# connect using our secret
$HOME/mongodb/bin/mongo -u super -p 'secret' --authenticationDatabase "admin"
# we want to create the user on the cid db, in generate users can be created in any
# db and provide access to multiple dbs, it's just the location where the user is stored
use cid
# if already exist
db.dropUser("crossid")
# Replace indexia with the relevant tenant id
# Replace _cidsecret_ with a strong password
db.createUser(
  {
    user: "crossid",
    pwd: "cidsecret",
    roles: [ {role: "readWrite", db: "indexia"}, {role: "readWrite", db: "cid"} ]
  }
)
```

At this point we can authenticate using the crossid user on the cid db that should be granted with sufficient privileges to manage the tenant and cid dbs.


# Enabling SSL


For production SSL is a must in order to secure the tunnel between the client and mongo.


It is highly recommended to sign a CSR by a trusted CA.


## Trusted CA

TODO


## Self Signed Certificate

Generate a self signed certificate by:

Note: The input for _Common Name_ must be the server name as written by clients connecting to the mongo daemon.


```bash
openssl req -newkey rsa:2048 -new -x509 -days 1095 -nodes -out $HOME/etc/mongod-cert.crt -keyout $HOME/etc/mongod-cert.key
cat $HOME/etc/mongod-cert.key $HOME/etc/mongod-cert.crt > $HOME/etc/mongod.pem
```

Enable the _net/ssl_ section in the _etc/mongod.conf_ file and restart the server.

At this point client is required to support SSL,

Try to connect via mongo CLI client:


```bash
$HOME/mongodb/bin/mongo -ssl --sslPEMKeyFile $HOME/etc/mongod.pem -sslCAFile $HOME/etc/mongod.pem -u crossid -p 'cidsecret' --authenticationDatabase "cid" serverName:27017/cid


### Mongod as a systemd service


As _root_ user run:

```bash
cp /home/mongod/pkg/mongod.service /etc/systemd/system
```

Enable the _systemLog_ and the _processManagement_ sections and run:

```bash
sudo systemctl daemon-reload
sudo systemctl start mongod
# should show service is active (running)
systemctl status mongod
```

# References

- https://docs.mongodb.com/manual/tutorial/configure-ssl/
- https://docs.mongodb.com/manual/tutorial/enable-authentication/
- https://docs.mongodb.com/manual/reference/configuration-options/
- https://docs.mongodb.com/manual/reference/connection-string/
  