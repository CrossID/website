---
title: Simple Vault
linktitle: Simlple Vault
description: Configuring the Simple Vault
date: 2019-07-27
publishdate: 2019-07-27
lastmod: 2019-07-27
categories: [security, vault]
keywords: []
menu:
  docs:
    parent: "Vault"
    weight: 1
weight: 1
draft: false
toc: false
---

The simple vault is an immutable secret storage and is the simplest way to store secrets.

It is embedded within CrossID and requires no integration.


- Secrets are encrypted in AES algorithm where passphrase is stored within a file.
- Secrets are stored in the tenant configuration.


## Tenant Configuration

```json
"vault" : {
  "type" : "simple",
  "params" : {
    "passphraseUri" : "file:///path/to/enc.key",
    "secrets" : {
      "/path/to/secret": "encrypted value"
    }
  }
}
```

- `passphraseUri` is the location of a file containing a single line of a random passphrase used to encrypt secrets.
- `secrets` is an object of multiple secrets in the form of _path_ -> _encrypted value_
  - _path_ can be arbitrary path in the form of _/nested/key_ (e.g., _"/secrets/apps/hr/credentials"_)
  - _encrypted value_ is the encrypted secret in base64 (see below)

## Creating a secret

A secret is a key/value pair nested under the _data_ property:

_secret.json_

```json
{
  "data": {
    "dbUser": "hiddenUser",
    "dbPassword": "topSecret"
  }
}
```

To encrypt the secret above run:

```bash
crossid secret put --path /secrets/apps/hr/credentials --file secret.json
```

This yields an output such:

```
Data: 
  - ciphertext: GDxwROIrsON9KEcm1YDjGksmu1mFx8s6hRT7=
```


Note: Since the _simple_ vault is immutable, the secret is only printed to the screen, copy paste the secret into the tenant _secrets_ as shown above.

To get the stored secret:

```bash
crossid secret get --path /secrets/apps/hr/credentials

Data: 
  - dbUser: hiddenUser
  - dbPassword: topSecret
```


## Securing Applications Credentials

To secure application credentials it is possible to refer to a stored secret within vault,

Consider this application manifest:

```json
{
  "appId": "hr",
  "appLogic": "rest",
  "displayName": "Human Resources",
  "config": {
    "user": "hiddenUser",
    "password": "topSecret"
  }
}
```

Instead of writing the credentials in clear text, we could refer to secrets in vault:

```json
{
  "appId": "hr",
  "appLogic": "rest",
  "displayName": "Human Resources",
  "config": {
    "user": "${vault://secrets/apps/hr/credentials?path=user}",
    "password": "${vault://secrets/apps/hr/credentials?path=password}",
  }
}
```

- Every secret is written in the form of `${vault://path/to/secret?path=key}`
- `/secrets/apps/hr/credentials` is the path where the secret is stored
- `?path=xxx` is the path to property _within the secret_.