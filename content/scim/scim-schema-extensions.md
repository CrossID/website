---
title: SCIM Schema Extensions
linktitle: SCIM Schema Extensions
description: What are SCIM schema extensions and how to use them
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

# Preface

SCIM Schema is a logical group of attributes,

Every resource type (such "User" or "Group") has a primary schema and possibly optional extensions.

The main schema defines how resources (e.g., "users") of some resource type (e.g., "User") look like,

For example the _urn:ietf:params:scim:schemas:core:2.0:User_ schema defines how a User should look like,

It is encouraged to use this core schema by every application that supports users,

But what happens when we need other attributes that are not covered by the primary schema? for instance, lets say we want to store the user's manager details,
since the manager is another user within the system, we want to store a reference to it, manager doesn't exist in the core User schema, thus we need an extension
that provides the definition of this attribute,

Luckily, SCIM comes with another schema named [enterprise](https://tools.ietf.org/html/rfc7643#section-4.3), which is an extension to the core user schema and can be used whenever organization attributes are needed,

# JSON representation of User with extension

SCIM distinguishes between the attributes of the primary schema and the extensions by placing the primary schema attributes in the root of the document while extension attributes are stored under the namespace of the extension.

Here is a JSON of a user with the enterprise extension:

{{< highlight json >}}

{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User", "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"],
  "id": "2819c223-7f76-453a-919d-413861904646",
  "externalId": "701984",
  "userName": "bjensen@example.com",
  "name": {
    "formatted": "Ms. Barbara J Jensen, III",
    "familyName": "Jensen",
    "givenName": "Barbara"
  }
  "displayName": "Babs Jensen",
  "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": {
    "employeeNumber": "701984",
    "costCenter": "4130",
    "organization": "Universal Studios",
    "division": "Theme Park",
    "department": "Tour Operations",
    "manager": {
      "value": "26118915-6090-4610-87e4-49d8ca9f808d",
      "$ref": "../Users/26118915-6090-4610-87e4-49d8ca9f808d",
      "displayName": "John Smith"
    }
  },
  "meta": {
    "resourceType": "User",
    ...
  }
}

{{< / highlight >}}


Lets explain the important attributes here:

_schemas_: Since this is a user record, it must have the _urn:ietf:params:scim:schemas:core:2.0:User_
but this user also have some enterprise attributes (e.g., "costCenter", "manager") it also have the _urn:ietf:params:scim:schemas:extension:enterprise:2.0:User_ schema.

Primary attributes (such _id_, _externalId_, etc) are defined in the root level of the record.

Please note: The extension schemas are required to be mentioned only if the record has attributes of the extension,
for example, some user records may not have the _enterprise_ schema if they don't don't store any of the enterprise attributes.

_urn:ietf:params:scim:schemas:extension:enterprise:2.0:User_: Every attribute of the _enterprise_ extension must be defined under the extension urn.

This way we can distinguish from attributes defined in the primary schema and attributes defined in the extensions.

