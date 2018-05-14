---
title: .Net Minion
linktitle: .Net Minion
description: Use the .NET Minion to integrate with any application that communicates easily using .NET Framework.
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
categories: [enterprise, onprem, private-cloud]
layout: single
menu:
  docs:
    parent: "enterprise"
    weight: 25
weight: 20
sections_weight: 20
draft: false
toc: true
---

When it comes to applications integrations, communication with applications may be easy in one technology while exhausting with another.
CrossID can integrate with applications using the [Java Minion](/enterprise/java-minion), the _.NET Minion_ or the _GoLang Minion_.

For example, the .NET Minion is used to communicate with applications such as _Active Directory_ or _Exchange_, Microsoft .NET APIs are much more native, robust and secure for such applications.

## Installation


### Pre Requisites

- The [Hardware Requirements](/enterprise/hardware-requirements) per environment.
- .Net 4.7, preferrably latest 4.7.x


### Unzipping the archive

The .NET Minion is shipped as a single archive, it is a standalone application and requires no container (such as IIS) to run.

Uncompress the _crossid-csminion-xx.zip_ into _c:\Program Files\Crossid CS Minion_

The service requires zero configuration to run, simply click the `crossid-csminion.exe` to ensure it starts properly, expect a message such:

`Server started and bound to address "http://*:9000/"`

At this point the minion service should be up and running.


### Configuration

The configuration is XML based and can be found in the `crossid-csminion.exe.config` file under the `<appSettings>` section.

| Property                | Description           |
|-------------------------|-----------------------|
|bindAddress|Base URL to bind the service to, in the format of _http(s)://ip:port/_, example: **https://\*:9000/** <br/><br/> note: _*_ can be used instead of ip to bind the service to all IPs.|
|serviceName|The name of the windows service. (See "Service Installation" below)|
|serviceDescription|The description of the windows service.|
|authJwtSecret|The secret to validate JWT tokens with, encoded in base64. <br/><br/>**Note: this must change in production.**|
|authJwtIssuer|Accept JWT tokens only for the given issuer, leave as is.|
|authJwtAudience|Accept JWT tokens only if the given audience matches, leave as is.|

### Service Installation

The minion can run by simply running the executable although the recommended approach is to install it as a _window service_ for benefits such: easy monitoring, auto restart, etc.

To install as a service:


1. Run: `C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe minion.exe`.
1. Open the _Services_ list, find the service, in the service properties change the credentials to the relevant user in the form of DOMAIN\username.

Note: For Kerberos based authentication for applications such _Active Directory_, make sure the service credentials has sufficient privileges to communicate with the applications.

### Logs

Every request and errors are captured and stored in files, the logs can be found in the _logs_ folder beneath the primary minion folder.

It is possible to change the logging verbosity by changing the _NLog.config_ file.

The possible logging levels are: _Trace_, _Debug_, _Info_, _Warn_, _Error_ and _Fatal_.

The levels are ordered by verbosity where _Trace_ is the most verbose where the default level is _Info_.

To increase the logging level to _Trace_, simply change the level of the line:

`<logger name="*" minlevel="Info" writeTo="f"/>`

to:

`<logger name="*" minlevel="Trace" writeTo="f"/>`


A restart is required for the change to take affect.


### Advanced

#### Issuing tokens

For debugging purposes, a JWT authentication token is required to be provided per request in order to communicate with the minion directly,

A token can be simply issued by running the following command on the backend:

```bash
crossid token generate --tenant-id <tenant> --sub-type minion --secret <secret> --exp 5000
```

Replace _\<tenant\>_ with the tenant id and _\<secret\>_ with the value set in the _authJwtSecret_ configuration parameter (not in base64 this time!)