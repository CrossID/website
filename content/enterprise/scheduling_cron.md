---
title: Scheduling commands with Cron
linktitle: Scheduling jobs with Cron
description: Explains how to schedule CrossID commands via Cron.
date: 2019-03-22
publishdate: 2019-03-22
lastmod: 2019-03-22
categories: [enterprise, onprem, private-cloud, schedule, cron]
layout: single
menu:
  docs:
    parent: "enterprise"
    weight: 20
weight: 30
sections_weight: 30
draft: false
toc: true
---

_Cron_ is a time-based job scheduler in Linux that can be used to schedule CrossID commands such app import, pinging apps, etc.

## Pre req

- CLI is configured with authentication, read to execute commands.


## Import 'hr' app every 3rd hour monday-friday

```bash
0 */3 * * 1-5 $HOME/bin/cid.sh app import --id hr >> $HOME/log/cron.log 2>&1
```

This command also pipe the output into _cron.log_ file for further investiagion.
