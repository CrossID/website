---
title: Authoritive Source (HR)
linktitle: Authoritive Source
description: Explaining how to define an authoritive source of people such HR system
date: 2018-01-09
publishdate: 2018-01-09
lastmod: 2018-01-09
categories: [scim, integration, cli]
layout: single
menu:
  docs:
    parent: "integration"
    weight: 20
weight: 20
sections_weight: 20
draft: false
toc: true
---

## Authoritive Source

CrossID is about people, in most organization there is some _authoritive source_ application that contain the list of people
that have access to the organizational applications, usually this is the HR system (e.g., _"Success Factor"_).

This tutorial is about how to connect the _authoritive source_ (for the rest of the tutorial lets name it "HR")

## What are we going to cover?

- Define the HR application, resource types and mappers.
- Map some of the HR app user attributes -> People attributes.
- Tell the import process to create a person for every new HR user.

## Define the HR application

To define an HR application we need at least:

1. _User_ resource type.
1. A mapper that maps HR's raw (the way attributes are defined in HR) attributes -> SCIM (the way attributes are defined in CID)
1. A _person mapper_ that maps some HR attributes -> person attributes.
1. Set the HR app as 'person creator' so every HR user will create a person (see below about this)

### Application manifest

{{< highlight json >}}
{
  "id": "7b9af5ba-e6d4-44e1-bc1e-d579579ff6e1",
  "tenantId": "dev",
  "appId": "hr",
  "appLogic": "myhr",
  "minion": "java-minion-host:9000",
  "displayName": "HR",
  "config": {
    "driver": "org.mariadb.jdbc.Driver",
    "url": "jdbc:mariadb://localhost:3306/hr",
    "userName": "root",
    "password": "pass",
    "validateQuery": "SELECT COUNT(*) from employees"
  }
}
{{< / highlight >}}


### User Resource Type

{{< highlight json >}}
{
  "id": "7f50382d-08bd-48b2-8acd-7af3afb84c9d",
  "name": "User",
  "description": "User type",
  "endpoint": "/Users",
  "schema": "urn:ietf:params:scim:schemas:core:2.0:User",
  "schemaExtensions": [
    { "schema": "crossid:common", "required": true},
    { "schema": "crossid:external", "required": true }
  ],
  "operations": {
    "search": {},
    "get": {}
  },
  "toSCIMMapper": "513c69ac-cdbd-47ee-bc8f-cf25b0f95da5"
  "meta": {
    "tenantId": "dev",
    "appId": "hr",
    "location": "/scim/v2/hr/ResourceTypes/7f50382d-08bd-48b2-8acd-7af3afb84c9d",
    "resourceType": "ResourceType"
  }
}
{{< / highlight >}}

### HR User 'to SCIM' mapper

This mapper is about mapping HR attributes into SCIM attributes,
The `user` object contains all _raw attributes_ (as they represented in HR).

{{< highlight json >}}
{
	"crossid:external": {
		"rawId": "user.EMP_ID"
	},
	userName: user.EMP_ID,
	displayName: "user.FIRST_NAME + ' ' + user.LAST_NAME",
	title: "user.POSITION_NAME",
	active: "!user.END_DATE",
	"name": {
		"givenName": "user.FIRST_NAME",
		"familyName": "user.FAMILY_NAME"
	},
	"userType": "user.COMPANY === 1 ? 'Employee' : 'Contractor'"
}
{{< / highlight >}}

Note: there is no need for the _from SCIM_ mapper since we don't update any HR data.


### Person Mapper

The person mapper maps HR user (in SCIM) -> the person attributes.

We map almost all attributes from HR user to the person since the HR user represent the person,
For other applications we map a very few attributes that we would like to have on the person.

Here's an example of a person mapper:

note: The `user` variable in the mapper is the HR user in SCIM format.

{{< highlight json >}}
{
	"userName": "user.userName",
	"active": "user.active",
	"displayName": "user.displayName",
	"name": {
		"givenName": "user.name.givenName",
		"familyName": "user.name.familyName"
	},
	"title": "user.title",
	"userType": "user.userType",
	"photos": [{value: 'http://peopleimages/' + user.userName}]
}
{{< / highlight >}}

### Important note about HR and Person _userName_ attribute.

Most HR apps at some point get the values of the directory (e.g., "AD") and email in the system, whether by some ETL reading it from some IDM system or directly from AD/Exchange.

Don't be tempted to use this AD / Email, the reason for this is becasue HR **is not the authoritive source of these attributes**, it just get it from somewhere else and
they'r not always there, especially for new people, in fact, in most cases CID is going to be responsbile for generating these values for new people so we can't
count on it.

TLTR;

So these attributes will be empty for every new person inserted in HR from the first place.

The flow is going to look like:

1. For all (most) existing people there will be AD/Email address in HR.
1. HR representators creates a new person in HR with **no** AD/Email.
1. CID discover the new record and create a local HR user and person in its DB with **no** AD/Email address.
1. CID starts a "new person workflow" and generate the userName / email address.
1. At some point CID or ETL will assign the userName / email into the HR's columns.

So if we map the HR AD username into person.userName this is what going to happen:

1. Most people will immediately have AD user in cidUser.userName as they already exist in HR.
1. HR representator creates a new person in HR with no AD / userName.
1. New Employee flow will start, generating the person's _userName_ (unless we store it somewhere else then we'll have dups)
1. HR sync will WIPE the person's _userName_ because it's still empty in HR.
1. HR will not be able to get the person's userName because it got wiped by the sync.

Conclusion: Even if we have user name in HR, don't tempt to map it to the person.

So what to use for HR / person _userName_ attribute ? use any unique identifier that is apart of the _core attributes_ of people that always exist and never change, such as employee number, or unique identifier that auto assigned by HR.

Please bare in mind: the most harden ID (that never change) is the better, in some orgs, some attributes such as "employeeNumber" may vary if the person is a contractor and becomes an employee, or may change if person leaves the company and re-join.

Note: You can still map the HR email / AD user column into some attribute such _externalId_ if you need to correlate the AD accounts for the initial correlation, but avoid using this attribute anywhere else.
