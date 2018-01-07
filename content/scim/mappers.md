---
title: SCIM Mappers
linktitle: SCIM Mappers
description: Explaining what are SCIM mappers
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
categories: [scim]
layout: single
menu:
  docs:
    parent: "scim"
    weight: 20
weight: 20
sections_weight: 20
draft: false
toc: true
---

While SCIM is well adopted by many cloud applications including GitHib, Slack and more, some applications have their own representation for resources such users and groups.

When importing data from external applications into CrossID, there is a need to map these external resources from their own data representation into SCIM standards,

When provisioning from CrossID into the application, there is a need for a reverse mapper that maps the SCIM representation of the user (as stored in CrossID) into the unique user representation as represented in the application.

This is where mappers come into play,

A mapper is a small JSON object that maps a resource type (such User or Group) into / from SCIM standards.


# To SCIM mapper

The _to scim_ mapper is used whenever CrossID imports data from external application.

Lets say we have a _dummy_ application with a user represented as follows:

{{< highlight json >}}
{
  "NATIONAL_ID": "1234",
  "USER_NAME": "mike_l",
  "FIRST_NAME": "Mike",
  "LAST_NAME": "Lander",
  "IS_ACTIVE": true,
  "GROUPS": [
  {
  	"id": "1-1-1-1",
  	"DESCRIPTION": "Project Admins"
  },
  {
  	"id": "1-1-1-2",
  	"DESCRIPTION": "Project Manager"
  }
  ]
}
{{< / highlight >}}


Lets walk through some of the User attributes:

**NATIONAL_ID**

a unique ID of the user


**USER_NAME**

user logs in with this user name

**IS_ACTIVE**

a true / false indication whether this user is active or not

**GROUPS**

an array of groups that the user belongs to, every group has a unique _id_ and some human _description_


In SCIM, Users have a [well known standard](what_is_scim) and CrossID stores the representation of any user in SCIM only,

We have to map each user attribute of our _dummy_ application into its corresponding SCIM attribute,

Here is an example of such mapper:

{{< highlight json >}}
{
  "crossid:common": {
    "rawId": "user.NATIONAL_ID"
  },
  "externalId": "user.NATIONAL_ID",
  "userName": "user.USER_NAME",
  "name": {
    "givenName": "user.FIRST_NAME",
    "familyName": "user.LAST_NAME"
  },
  "active": "user.IS_ACTIVE",
  "groups": {
    "_multi": "user.GROUPS",
    "value": "it.id"
  }
}
{{< / highlight >}}


Lets explain this mapper:

The keys of the JSON object represent the outcome of the mapping process, this is how the user is going to look like in the SCIM form,
The value is a javascript expression with one variable named _user_, this _user_ variable contains all the attributes of the user as represented in the dummy application.

In SCIM the following attributes means:

**crossid:common.rawId**

rawId is a sepcial (non SCIM) required attribute that must contain the unique identifier of the user in the application, this is why it is put under the _crossid:common_ extension. This attribute expect to have a value that never changes through the entire lifetime of the user.

**externalId**

the identifier of the user as represented in the application


**userName**

user logs in with this user name

**name**

nested object with person name

**active**

a true / false indication whether this user is active or not

**groups**

an array of groups that the user belongs to, please note that CrossID only expect one attribute per object named _value_ that contains the unique ID of the group


Please note that the _"_multi": "user.GROUPS"_ is a special hint for the mapper and is needed whenever we deal with mapping an array of objects.


The outcome of such mapping process will yield the following result:

{{< highlight json >}}
{
  "crossid:common": {
    "rawId": "1234"
  },
  "externalId": "1234",
  "userName": "mike_l",
  "name": {
    "givenName": "Mike",
    "familyName": "Lander",
  },
  "active": true,
  "groups": [
    {
      "value": "1-1-1-1"
    },
    {
      "id": "1-1-1-2"
    }
  ]
}
{{< / highlight >}}


# From SCIM mapper

CrossID stores every resource in its SCIM representation, that's why we need the _To SCIM_ mapper,

Since the external application doesn't understand SCIM, whenever CrossID provosion resources into the application it needs to map the SCIM attributes back into the repesentation of the user as represented in the application.

This is where the _From SCIM_ mapper comes in play,

Lets take a look at such mapper:

TODO