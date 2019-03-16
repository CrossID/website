---
title: Resources Mapping
description: Map attributes of one resource to another.
date: 2018-07-25
publishdate: 2018-07-25
lastmod: 2018-07-25
categories: [integration]
layout: single
menu:
  docs:
    parent: "integration"
    weight: 30
weight: 30
sections_weight: 20
draft: false
toc: true
---

# How to map AD to CID

This guid explains how to map the _userPrincipalName_ attribute from _Azure_ user to a _CrossID_ user.

This is required for authentication to work properly.

Pre requisites:

- The _CrossID_ user shell be the owner of the _Active Directory_ user.


# Mapping Command

```bash
crossid resource map --filter 'meta.appId eq "cid" and meta.resourceType eq "User" and active eq true' --corr-id 19bd0e5e-8a49-43ce-8ecb-17bcf9e0dd55 --mapper-id 6ec1f9ee-7074-4567-b8fc-f8668d445878
```

Lets explain this command:

1. `--filter` finds all the targeted cid users (aka: "Target") we would like to map the attributes TO.
1. `--corr-id` is an ID of a filter model that tries to match an _Azure_ user (aka: "Source") to every targeted cid user. (resulted by `--filter`)
1. `--mapper-id` is an ID of a mapper model that maps data from _Source_ to _Target_ per match.


# Correlation Filter

The following correlation filter will run for every user that was matched by the filter above:


```json
{
  "id": "19bd0e5e-8a49-43ce-8ecb-17bcf9e0dd55",
  "name": "Correlate AD user to a CID user",
  "description": "In context we have an Azure user, we try to find a matching CID user.",
  "table": "correlations",
  "condition": "userName eq {user.externalId} and meta.resourceType eq \"User\" and meta.appId eq \"cid\"",
  "template": true
}
```

- The _{user.externalId}_ will be replaced with the CID user in context and the query will be invoked.
- If an _Azure_ user was matched, it will be used by the mapper below.

# Mapper

This mapper will run for every tuple of matched CID and AD users: 


```json
{
  "id": "",
  "expType": "js",
  "mapper": {
    "urn:crossid:auth": {
      "userPrincipalName": "get(user, 'userPrincipalName')"
    }
  }
}
```

- The _user_ in context is the _Azure_ user.
- The attributes yielded by the mapper will be merged with the matched CID user.
