---
title: Hardware Requirements
linktitle: Hardware Requirements
description: Product hardware requirements for enterprise installation.
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

This document describes the hardware requirements for CrossID installation.

The requirements applicable for SMB companies (up to 6000 employees with up to 100 applications), please contact us for more custom demands if necessary.

## Production

Description of production grade environment.

### Backend

Following are the requirements for the primary backend server.

| Property                | Minimum               | Recommended           |
|-------------------------|-----------------------|-----------------------|
| Virtualization Platform | ESX only              | ESX only              |
| OS                      | Linux RHEL 7.4        | Linux RHEL 7.4        |
| RAM                     | 16 GB                 | 32 GB                 |
| Storage                 | 100GB                 | 250GB                 |
| CPU                     | 4 Xeon VCPUs, 3.xGHz+ | 8 Xeon VCPUs, 3.xGHz+ |

### Java Minion

This java minion is used to communicate with on prem protocols such: databases or web services.


| Property                | Minimum               | Recommended           |
|-------------------------|-----------------------|-----------------------|
| Virtualization Platform | ESX only              | ESX only              |
| OS                      | Linux RHEL 7.4        | Linux RHEL 7.4        |
| RAM                     | 8 GB                  | 16 GB                 |
| Storage                 | 40GB                  | 80GB                  |
| CPU                     | 4 Xeon VCPUs, 3.xGHz+ | 8 Xeon VCPUs, 3.xGHz+ |


### .NET Minion

This is a server used to communicate with Microsoft applications such as Active Directory or Exchange.

| Property                | Minimum                                  | Recommended                            |
|-------------------------|------------------------------------------|----------------------------------------|
| Virtualization Platform | ESX only / Hypervisor                    | ESX only / Hypervisor                  |
| OS                      | Windows Server 12 / 16                   | Windows Server 12 / 16                 |
| RAM                     | 8 GB                                     | 16 GB                                  |
| Storage                 | 40GB                                     | 80GB                                   |
| CPU                     | 4 Xeon VCPUs, 3.xGHz+                    | 8 Xeon VCPUs, 3.xGHz+                  |
| Extra Softwares         | .NET 4.7                                 | .NET 4.7                               |
| Domain                  | Member of the AD domain to be managed.   | Member of the AD domain to be managed. |


## Test

Description of POC, tests, QA, staging environments.

### Backend

Following are the requirements for the primary backend server.

| Property                | Minimum               | Recommended           |
|-------------------------|-----------------------|-----------------------|
| Virtualization Platform | ESX only              | ESX only              |
| OS                      | Linux RHEL 7.4        | Linux RHEL 7.4        |
| RAM                     | 8 GB                  | 16 GB                 |
| Storage                 | 50GB                  | 80GB                  |
| CPU                     | 2 Xeon VCPUs, 3.xGHz+ | 4 Xeon VCPUs, 3.xGHz+ |

### Java Minion

This java minion is used to communicate with on prem protocols such: databases or web services.


| Property                | Minimum               | Recommended           |
|-------------------------|-----------------------|-----------------------|
| Virtualization Platform | ESX only              | ESX only              |
| OS                      | Linux RHEL 7.4        | Linux RHEL 7.4        |
| RAM                     | 8 GB                  | 8 GB                  |
| Storage                 | 40GB                  | 60GB                  |
| CPU                     | 2 Xeon VCPUs, 3.xGHz+ | 2 Xeon VCPUs, 3.xGHz+ |


### .NET Minion

This is a server used to communicate with Microsoft applications such as Active Directory or Exchange.

| Property                | Minimum                                  | Recommended                            |
|-------------------------|------------------------------------------|----------------------------------------|
| Virtualization Platform | ESX only / Hypervisor                    | ESX only / Hypervisor                  |
| OS                      | Windows Server 12 / 16                   | Windows Server 12 / 16                 |
| RAM                     | 8 GB                                     | 12 GB                                  |
| Storage                 | 40GB                                     | 60GB                                   |
| CPU                     | 2 Xeon VCPUs, 3.xGHz+                    | 2 Xeon VCPUs, 3.xGHz+                  |
| Extra Softwares         | .NET 4.7                                 | .NET 4.7                               |
| Domain                  | Member of the AD domain to be managed.   | Member of the AD domain to be managed. |


## RDBMS

The relational database is used by the business process engine.

| Property                | Minimum                                      | Recommended                                  |
|-------------------------|----------------------------------------------|----------------------------------------------|
| Database                | Oracle 12c or MSSQL Server 2008 R2/2012/2014 | Oracle 12c or MSSQL Server 2008 R2/2012/2014 |
| Schema Name             | crossid_prod or crossid_test                 | crossid_prod or crossid_test                 |
| Estimated Size          | 16 GB                                        | 16 GB                                        |
