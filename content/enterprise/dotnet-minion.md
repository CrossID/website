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

Uncompress the _crossid-csminion-xx.zip_ into _c:\Program Files\CrossID .NET Minion_

The service requires zero configuration to run, simply click the `crossid-csminion.exe` to ensure it starts properly, expect a message such:

`Server started and bound to address "http://*:9000/"`

At this point the minion service should be up and running.


### Configuration

The configuration is XML based and can be found in the `crossid-csminion.exe.config` file under the `<appSettings>` section.

| Property                | Category|  Description    |
|-------------------------|---------|-----------------|
|bindAddress|Network|Base URL to bind the service to, in the format of _http(s)://ip:port/_, example: **https://\*:9000/** <br/><br/> note: _*_ can be used instead of ip to bind the service to all IPs.|
|authJwtSecret|Security|The secret to validate JWT tokens with, encoded in base64. <br/><br/>**Note: this must change in production.**|
|authJwtIssuer|Security|Accept JWT tokens only for the given issuer, leave as is.|
|authJwtAudience|Security|Accept JWT tokens only if the given audience matches, leave as is.|
|authAllowedSingleIPs|Security|coma separated IP addresses to restrict connections from.|
|serilog:minimum-level|Logging|The minimal logging level to write, possible values are: _Verbose_, _Debug_, _Information_, _Warning_, _Error_ and _Fatal_|
|serilog:write-to:File.path|Logging|The path where logs are stored, (e.g., _C:\Users\Administrator\AppData\Roaming\CrossID .NET Minion\logs_)|

Note: Any configuration property not mentioned here should not be modified.

### Service Installation

The minion can run by simply running the executable although the recommended approach is to install it as a _window service_ for benefits such: easy monitoring, auto restart, etc.

To install as a service:

1. `cd C:\Program Files\CrossID .NET Minion`
1. Run: `InstallUtil.exe crossid-csminion.exe`.
1. Supply user and password (please note that user should be in the form of `DOMAIN\Username` (e.g., "CROSSID\CidAdmin"))
1. Open the _Services_ list, find the service, in the service properties change the credentials to the relevant user in the form of DOMAIN\username.

Note: InstallUtil.exe should be located under `C:\Windows\Microsoft.NET\Framework\v4.0.30319`

Note: For Kerberos based authentication for applications such _Active Directory_, make sure the service credentials has sufficient privileges to communicate with the applications.


To uninstall the service run `InstallUtil.exe crossid-csminion.exe /u`


### Enable SSL

SSL requires binding a certificate to a port.

Import the certificate into `LOCAL COMPUTER > Personal > Certificates`

run `dir cert:\localmachine\my` to get the _Thumbprint_

then run:

```powershell
$guid = [guid]::NewGuid()
$certHash = "09E7C2609A442057BB3D8170B8F0FA8F4806F298" # replace with the correct Thumbprint
$ip = "0.0.0.0"
$port = 9000
"http add sslcert ipport=$($ip):$port certhash=$certHash appid={$guid}" | netsh
```
expect _SSL Certificate successfully added_, now every connection attempt to that ip/port would be answered using the specified certificate.



### Advanced

#### Issuing tokens

For debugging purposes, a JWT authentication token is required to be provided per request in order to communicate with the minion directly,

A token can be simply issued by running the following command on the backend:

```bash
crossid token generate --tenant-id <tenant> --sub-type minion --secret <secret> --exp 5000
```

Replace _\<tenant\>_ with the tenant id and _\<secret\>_ with the value set in the _authJwtSecret_ configuration parameter (not in base64 this time!)
