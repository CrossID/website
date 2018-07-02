---
title: Store
linktitle: Store
description: Where backend stores its data
date: 2018-07-02
publishdate: 2018-07-02
lastmod: 2018-07-02
categories: [enterprise, onprem, private-cloud]
layout: single
menu:
  docs:
    parent: "enterprise"
    weight: 20
weight: 30
sections_weight: 20
draft: false
toc: true
---

The store is where backend stores its objects including resources imported from applications or internal models such _rules_, _jobs_, etc.

This document defines the store requirements in an enterprise setup per supported store type.


## Mongo

### Configuration DB

A tiny DB where backend stores cross tenants data such tenant meta.

| Property                | Value            | Notes                                                        |
|-------------------------|------------------|--------------------------------------------------------------|
| DB Name                 | cid              |                                                              |
| Storage                 | 1MB              | size won't increase over time                                |
| SSL                     | enabled          | optional but recommended                                     |
| Username                | _any_            |                                                              |
| Password                | _any_            |                                                              |


### Tenant Specific DB

The tenant specific DB is where all data is stored for a given tenant.

Such DB should be created per tenant configured in the backend.

| Property                | Value            | Notes                                                                              |
|-------------------------|------------------|------------------------------------------------------------------------------------|
| DB Name                 | _\<tenant name\>_| name must be equal to the _tenant id_ set in the backend                           |
| Storage                 | 3GB+             | size may vary and increase over time based on amount of applications and resources |
| SSL                     | enabled          | optional but recommended                                                           |
| Username                | _any_            |                                                                                    |
| Password                | _any_            |                                                                                    |


CrossID uses MongoDB _oplog_ as an event source, please make sure MongoDB is configured to produce an oplog.

The oplog can be enabled by using one of the following options:

1. Setting up [replica sets](https://docs.mongodb.com/manual/tutorial/deploy-replica-set)
1. Passing --master to the mongod process
1. Setting `master = true` in /etc/mongod.conf
