---
title: Manual Camunda BPMN Setup
linktitle: Manual Camunda BPMN Setup
description: How to manually setup MongoDB
date: 2018-11-23
publishdate: 2018-11-23
lastmod: 2018-12-12
categories: [enterprise, onprem, private-cloud, setup, installation, manual, bpmn, workflow]
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

This page explains how to setup Camunda business process engine manually,

It is intended for those who can't or don't want to use an automated approach such docker or Ansible book.


# Pre requisites

- Linux x64 (such RHEL 7.x) with root / sudo access.
- Recommended: external RDBS such Oracle or MSSQL.
- JRE 1.8.x installed
- [Camunda Tomcat distribution zip](https://camunda.com/download/)

# Setup layout

Our folder layout will look like:

```
pkg - installation files
```

# Basics

```bash
# Change to the relevant version based on the zip file
export CAMUNDA_VER=7.9.0
useradd camunda
passwd camunda
su - camunda
mkdir -p $HOME/pkg
```

# Manually copy installation files

Manually copy the zip file into `~/pkg` folder

## Extract Archive

```bash
mkdir -p camunda-${CAMUNDA_VER}
ln -s camunda-${CAMUNDA_VER} camunda
unzip pkg/camunda-bpm-tomcat-${CAMUNDA_VER}.zip -d camunda
# cleanup samples
rm -rf ~/camunda/server/apache-tomcat-9.x.x/webapps/camunda-invoice
rm -rf ~/camunda/server/apache-tomcat-9.x.x/webapps/camunda-welcome
rm -rf ~/camunda/server/apache-tomcat-9.x.x/webapps/examples
rm -rf ~/camunda/server/apache-tomcat-9.x.x/webapps/docs
rm -rf ~/camunda/server/apache-tomcat-9.x.x/webapps/h2
cd ~/camunda
```

## Setup Network

Change port from 8080 to 9090 to avoid collisions.

vi _~camunda/camunda/server/apache-tomcat-9.x.x/conf/server.xml_


```xml
 <Connector port="9090" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="9443" />
```

# External DB

For production it is highly recommended to use a DB such Oracle or MSSQL.

## Oracle

Manually copy [ojdbcX.jar](https://www.oracle.com/technetwork/database/application-development/jdbc/downloads/index.html) to _~camunda/camunda/server/apache-tomcat-9.x.x/lib_


Edit _camunda/server/apache-tomcat-9.x.x/conf/server.xml_ and replace existing Tomcat _jdbc/ProcessEngine_ datasource with the following:

```xml
<Resource name="jdbc/ProcessEngine"
              auth="Container"
              type="javax.sql.DataSource" 
              factory="org.apache.tomcat.jdbc.pool.DataSourceFactory"
              uniqueResourceName="process-engine"
              driverClassName="oracle.jdbc.OracleDriver" 
              url="jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=server.corp)(PORT=1521))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=serviceName)))"
              defaultTransactionIsolation="READ_COMMITTED"
              username="CID_BP"
              password="password"
              maxActive="20"
              minIdle="5"
              maxIdle="20" />
```


## Cleanup H2 DB

```bash
rm -rf ~/camunda/server/apache-tomcat-9.x.x/webapps/h2
```


# Configure Date Format

Camunda must be configured to use ISO3339 as its date format to conform CrossID standards.


Edit file _camunda/server/apache-tomcat-9.x.x/webapps/engine-rest/WEB-INF/web.xml_ and add the _listener_ and _context-param_ tags:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

  <!-- ... -->

  <listener>
    <listener-class>
      org.camunda.bpm.engine.rest.CustomJacksonDateFormatListener
    </listener-class>
  </listener>

  <context-param>
    <param-name>org.camunda.bpm.engine.rest.jackson.dateFormat</param-name>
    <param-value>yyyy-MM-dd'T'HH:mm:ssXXX</param-value>
  </context-param>

  <!-- ... -->
</web-app>
```


# Start the server

```bash
cd ~
./camunda/start-camunda.sh
```

visit: http://server:9090/camunda

create user name _crossid_ with strong password.

create tenant, to avoid confusions name it the same as the CrossID tenant (e.g., "lab", "prod", etc.)

Go to users -> List of users and select _Crossid_ user and assign it to the tenant.

Following command should result an empty array with status code 200:

```bash
curl http://localhost:9090/engine-rest/process-definition -v
```


# Enable authentication

```bash
vi ~/camunda/server/apache-tomcat-9.x.x/webapps/engine-rest/WEB-INF/web.xml
```

Enable the commented out _Http Basic Authentication Filter_ XML tag.

restart camunda by:

```bash
cd ~
./camunda/server/apache-tomcat-9.0.5/bin/shutdown.sh
./camunda/start-camunda.sh
# once started, the following should result status code 401
curl http://localhost:9090/engine-rest/process-definition -v
# should show empty array with status code 200
curl http://localhost:9090/engine-rest/process-definition -u crossid:<password> -v
```



# TLS

## Tomcat

- Firewall should block any traffic to Camunda _9090_ port except proxy / backend.
- Enable TLS on Tomact (currently not documented)


## Via local Proxy

- If camunda runs on the same server where backend installed then it is higly recommended to configure the Connector (see _Setup Network_ above) to be bound to localhost only by adding `address="127.0.0.1"` param.
- Configure proxy to pass traffic of path _"/camunda"_ to _127.0.0.1_


### Traefik Proxy

Add the following backend:

```yaml
[backends.camunda]
  [backends.camunda.servers.server1]
  url = "http://server:9090"
```

Add the following frontend:

```yaml
[frontends.camunda]
backend = "camunda"
  [frontends.camunda.routes.camunda]
  rule = "PathPrefix:/camunda"
```


# Troubleshooting

- Logs location: _~/camunda/server/apache-tomcat-9.x.x/logs/catalina.out_



# References

- [BPMN Config File Reference](https://docs.camunda.org/manual/7.9/reference/deployment-descriptors/descriptors/bpm-platform-xml/)