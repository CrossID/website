---
title: What is SCIM
linktitle: What is SCIM
description: What is SCIM and how it is used in CrossID
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
layout: single
menu:
  docs:
    parent: "scim"
    weight: 10
weight: 10
sections_weight: 10
draft: false
toc: true
---

SCIM is an acronym for _System for Cross-domain Identity Management_, it is an open standard that defines how identity information (such users and group)
look like (the schema) and how to exchange (e.g., "add", "delete") these identities across domains (the protocol).

It aims to simplify identity provisioning and operational management in cloud services.


SCIM is divided into two main sections:

1. Describe how resources such _Users_ and _Groups_ look like.
1. Define how resources should be exchanged across domains using a known RESTful.

# How common resource types look like in SCIM?

SCIM defines the canonical schema of _Users_ and _Groups_ as these are the most common resource types across applications,

The User and Group core schemas dictates how users and group should look like and their behavors such possible attriubte names, their type and whether they are required or not.

Note: It is possible to define other resource types (e.g., "Roles" or "Profiles") if some application supports.

## User representation

The core schema for "User" is identified using the following schema URI: _"urn:ietf:params:scim:schemas:core:2.0:User"_,

Here is example of minimalist representation of a user:


{{< highlight json >}}
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "id": "2819c223-7f76-453a-919d-413861904646",
  "userName": "bjensen@example.com",
  "meta": {
    "resourceType": "User",
    "created": "2010-01-23T04:56:22Z",
    "lastModified": "2011-05-13T04:42:34Z",
    "version": "W\/\"3694e05e9dff590\"",
    "location": "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646"
  }
}
{{< / highlight >}}

Lets example these attributes and their meaning:

`schemas` - an array of schema IDs that defines this JSON, when dealing with users, it value will always be the presented core schema plus optional other extensions.
`id` - unique identifier of the resource
`userName` - unique identifier for the user, usually used by the person that owns it to directly authenticate to the app.
`meta` - some meta data about this resource, please see [Common Attributes] for more info

The [core user schema](https://tools.ietf.org/html/rfc7643#section-8.7.1) dictates that these are the only required attributes while there are many other optional attributes,

User is very detailed in SCIM, take a look at its [full user representation](https://tools.ietf.org/html/rfc7643#section-8.2) for all possible attributes.

## Group representation

Group is much simpler and contains very small amount of attributes, here is a full group representation:

{{< highlight json >}}
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
   "id": "e9e30dba-f08f-4109-8486-d5c6a331660a",
   "displayName": "Tour Guides",
   "members": [
     {
       "value": "2819c223-7f76-453a-919d-413861904646",
       "$ref": "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646",
       "display": "Babs Jensen"
     },
     {
       "value": "902c246b-6245-4190-8e05-00816be7344a",
       "$ref": "https://example.com/v2/Users/902c246b-6245-4190-8e05-00816be7344a",
       "display": "Mandy Pepperidge"
     }
   ],
   "meta": {
     "resourceType": "Group",
     "created": "2010-01-23T04:56:22Z",
     "lastModified": "2011-05-13T04:42:34Z",
     "version": "W\/\"3694e05e9dff592\"",
     "location": "https://example.com/v2/Groups/e9e30dba-f08f-4109-8486-d5c6a331660a"
  }
}
{{< / highlight >}}


_members_ is a multi valued attribute where each element represents a reference to some user, members can also refer to another group when dealing with nested group membership.

In SCIM, in most cases an object (complex type) that describes a reference has the following attributes:

_value_ - the unique id of the refered resource.
_$ref_ - a full URL of the refered resource, since SCIM protocl is RESTful, when submitting a GET request to this URL the expected result would be the representation of the referenced URL.
_display_ - The display name of the refered resource.


As with any resource type, it is possible to extend the basic attributes with [schema extensions](scim_schema_extensions) whenever extra attributes are needed.

