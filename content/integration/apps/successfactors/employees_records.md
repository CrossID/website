---
title: Listing employee records
linktitle: Listing employee records
description: This document explains how to list employee records
date: 2019-03-11
publishdate: 2019-03-11
lastmod: 2019-03-11
categories: [integration, sap, cloud, success-factors]
layout: single
weight: 20
sections_weight: 20
draft: false
toc: true
---


# Preface

The purpose of this document is to explain how employee records are read from SAP SuccessFactors (aka: "SF") for CrossID integration.

SF maintains _startDate_ and _endDate_ for employee records, including expanded navigation entities (e.g., "jobInfoNav").

The complexity is raised because CrossID does not maintain start/end date of every piece of data,

The requirement is to import all records from SF into CrossID constantly.


1. All records should be listed, including future records (e.g., "an employee is inserted into SF with a future startDate, where nav entities are in the future").
2. Changes in the near future (e.g., "14 days from current") shell optionally be peeked. Peeking into a near future (e.g., "14 days from current date") of how employees records would look (including navigation entities) is required to provide a sufficient time frame for helpdesk to take care of employee arrangements regarding to identity management prior to the employement change. For example, if an employee job information is appended with startDate: _2019-04-20_, the new job should be picked (instead of current one) at
date _2019-04-06_.


# Querying dates

- SAP, by default returns data for today (same as if _asOfDate_ query param is set to _sysdate_).
- _asOfDate_ cannot be specified in conjuction with _fromDate_ and/or _toDate_ query params.
- Setting the _fromDate_ query param means that SAP will return records where navigation entities endDate is greater or equal to the provided _fromDate_ value 
 (does not take into consideration the _startDate_ of records, see _toDate_)
- Setting the _toDate_ query param means that SAP will return records where startDate is lower than or equal to the provided _toDate_ value.
- Simulations below use sysdate() where its value is the _2019-03-10_.


# References

- https://testhelpportal.com/viewer/b2b06831c2cb4d5facd1dfde49a7aab5/1811/en-US/e6b758ef7a744f75aee7ebf7e5c5fe62.html


# Q&A

- Why we can't just use _asOfDate_ to sysdate + 14 days ? because some records may be inserted into a future dates (such as two months ahead)
  Setting _asOfDate_ for a very future date (e.g., "a year from today") will cause all navigations (such JobInfo) to show futured positions only.
  Which will eliminate the possibility to see current positions (see _change of position_ flow)


# Hire Flow

Hire flow inserts a person that was never employed before as a new employee record.

## Present

A person is hired where _startDate_ is lower than or equal to today.

- With no date query params, SAP returns the data for today (_asOfDate_=sysdate and _fromDate_, _toDate_ are not set),
This will cause all data of such record in API Call to be retrieved.

- If _fromDate_=sysdate() is set, full data will be returned since the _endDate_ of the navigation entities will always be set to 9999-12-31 
(see _fromDate_ logic above) this behavior is equal as with absence of this query param.

### Simulation:

Assumptions:

- A new person is hired with startDate of '2019-03-10'.

Record will look like (assuming _fromDate_ is set to sysdate()):

```json
{
  "startDate": "2019-03-10",
  "endDate": null,
  "personalInfoNav": [
    {
      "startDate": "2019-03-10",
      "endDate": "9999-31-12"
    }
  ],
  "jobInfoNav":[
    {
      "startDate": "2019-03-10",
      "endDate": "9999-31-12"
    }
  ]
}
```

## Future

A person is hired with a future start date, in this flow an employee record is inserted into SAP on submission date but its attributes
and its navigation entities (e.g., "personalInfoNav", "jobInfoNav") have future start dates.

- With no date query params, SAP returns the data for today, even though the record itself has a future date -- it is returned,
but all navigation entities are not, that means data such firstName (apart of _personalInfoNav_) or organisational details (apart of _jobInfoNav_) won't be seen.
Data will become visible when sysdate >= startDate.

QUESTION: If record itself has future startDate, why it is returned if no date query params are set?

- If _fromDate_=sysdate() is set, full data is returned since all navigation entities has time frame such:
startDate: some-date-in-the-near-future
endDate: 9999 / null

The condition above will fetch these future attributes, making API to retrieve any future data:

### Simulation

Assumptions:

- A new person is hired with startDate of '2020-03-10'.

Record will look like (assuming _fromDate_ is set to sysdate()):

```json
{
  "startDate": "2020-03-10",
  "endDate": null,
  "personalInfoNav": [
    {
      "startDate": "2020-03-10",
      "endDate": null
    }
  ],
  "jobInfoNav":[
    {
      "startDate": "2020-03-10",
      "endDate": null
    }
  ]
}
```

# Termination

Termination is a flow where an existing (hired) employee's engagement with the company is to be terminated.

During termination flow, the following details are modified:

- endDate is set to the termination date (present or future).
- another jobInfo record is inserted.

few notes about API behavior:

- both if _fromDate_ is set / not set, the employee record _endDate_ change is reflected immediately in the API.

## Simulation

- Employee _startDate_ is _2016-09-25_ where today is the _2019-03-10_ and termination date is set to _2019-03-29_
- _fromDate_ is set to sysdate()

The record would look like:

```json
{
  "startDate": "2016-09-25",
  "endDate": "2019-03-29",
  "personalInfoNav": [
    {
      "startDate": "2016-09-25",
      "endDate": "9999-12-31"
    }
  ],
  "jobInfoNav": [
    {
      "startDate": "2018-01-10",
      "endDate": "2019-04-15",
      "jobTitle": "CTO",
      "eventReason": "Employee-Outsource"
    },
    {
      "startDate": "2019-03-30",
      "endDate": "9999-31-12",
      "jobTitle": "CTO",
      "eventReason": "TERM"
    }
  ]
}
```


# Re-hire

Re-hire is a flow where a person that was employed before and then was terminated is re-hired to the corporate.

Since SAP does not delete records, the existing record of the person gets modified,
Following are some of the changed attributes on the re-hired employee record after re-hire flow is completed:

- startDate is set to present / future date according to the hire contract.
- endDate is removed (set to null)
- navigation entities (such "personalInfoNav" or "jobInfoNav") are apended with another record (in conjuction with the previous navigation entities)

## Simulation

- The re-hired employee previous startDate was the _2016-03-15_.
- The re-hired employee previous endDate was the _2018-03-10_.
- New startDate is set to future date _2020-05-10_
- _fromDate_ is not set.

The record would look like:

```json
{
  "startDate": "2020-05-10",
  "endDate": "null",
  "personalInfoNav": [
    {
      "startDate": "2016-03-15",
      "endDate": "2018-03-10"
    },
    {
      "startDate": "2020-05-10",
      "endDate": "9999-12-31"
    },
  ],
  "jobInfoNav":[
    {
      "startDate": "2016-03-15",
      "endDate": "2018-03-10"
    },
    {
      "startDate": "2020-05-10",
      "endDate": "9999-12-31"
    }
  ]
}
```

personalInfoNav / jobInfo navs:

The employee has TWO objects (ordered by)
[0] the previous job info prior to re-hire (startDate = start date of employement, endDate = end date of employement)
[1] the futured one (statDate: same as start date after re-hire, no end date)


If _fromDate_ is set to sysdate() (2019-03-10) then record would look like:

```json
{
  "startDate": "2020-05-10",
  "endDate": "null",
  "personalInfoNav": [
    {
      "startDate": "2020-05-10",
      "endDate": "9999-12-31"
    },
  ],
  "jobInfoNav":[
    {
      "startDate": "2020-05-10",
      "endDate": "9999-12-31"
    }
  ]
}
```


# Change position

Change position is a flow where an existing employee is transitioned to another organizational position during its engagement with the company.

The job information (jobInfoNav) of the employee changes once flow is completed.


## Present


- Employee position is changed for today (_2019-03-10_)
- _fromDate_ is not set / _fromDate_ is set for today (_2019-03-10_)


The record would look like:

```json
{
  "jobInfoNav":[
    {
      "startDate": "2019-03-10",
      "endDate": "9999-12-31"
    }
  ]
}
```


## Future

- Employee position is changed for future date (e.g., "2020-03-15").
- _fromDate_ is not set.


The record would look like:


```json
{
  "jobInfoNav":[
    {
      "startDate": "2019-02-01",
      "endDate": "2020-03-14"
    }
  ]
}
```


- _fromDate_ is set for today (_2019-03-10_)

```json
{
  "jobInfoNav":[
    {
      "startDate": "2019-02-01",
      "endDate": "2020-03-14"
    },
    {
      "startDate": "2020-03-15",
      "endDate": "9999-12-31"
    }
  ]
}
```


# Logic for peeking data in the near future

Following is the logic in order to peek data in the near future (e.g., "14 days" ahead):


- Listing employee records via OData API with setting _fromDate_=sysdate().
- Any present/future (no time limitation) hire records will be included.
- Will always return one record per employee.
- Expanded nav entities (e.g., "pesonalInfoNav", "jobInfoNav") may return with multiple records.
- If one nav entity is returned then logic will choose this one.
- If multi nav entities are returned, the logic will set _futureDate_ var = sysdate() + 14 days. logic will find the entity that matches the time frame of the _futureDate_ variable.

