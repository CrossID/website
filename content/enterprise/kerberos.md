---
title: Kerberos Authentication Configuration using Active Directory
linktitle: Kerberos Authentication using Active Directory
description: How to configure backend to authenticate via GSSAPI Kerberos (SPNEGO) protocol using Active Directory
date: 2018-03-27
publishdate: 2018-03-27
lastmod: 2018-03-27
categories: [enterprise, onprem, private-cloud, active-directory, gssapi]
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

Kerberos is a best practice internal SSO authentication protocol in many organizations,

CrossID supports GSSAPI Negotiation Mechanism (SPNEGO) to enable single sign-on (SSO) when using Active Directory as the KDC in a Windows environment. When SSO is enabled, users are logged in silently without any interaction.

This guide explains how to configure CrossID backend to authenticate users silentely (SSO) by using the Kerberos protocol when the KDC is Active Directory.

## Requirements

- Active Directory 2008 or 2016.
- Decide which domain to use to enable SSO (in this example we use the _INDEXIA.CO_ domain)
- Administrative access to the Active Directory domain.
- Users workstations must be logged into the chosen domain for the auth process.
- In Active Directory, create a user who represents the crossid backend (see _Creating a user in Active Directory_ below)
- Map the service principal name (see _Create SPN and Keytab_ below)

## Creating a user in Active Directory

Register a service as a user in Active Directory on the domain controller to represent CrossID. Open _Active Directory Users And Computers_ and add a new user.

![Create User 1](/sshots/ad_auth/s1_create_user.png)

![Create User 2](/sshots/ad_auth/s2_create_user_props.png)

![Create User 3](/sshots/ad_auth/s3_create_user_enc_types.png)

Please make sure to check the relevant properties as shown in the screenshots.


## Create SPN and Keytab

Map an SPN to the created user:

`ktpass /princ HTTP/<host>@<REALM> /ptype KRB5_NT_PRINCIPAL /crypto AES128-SHA1 /mapuser <user> /out <file>`

Provided values are described as follows:

**`host`**: Fully qualified name of the CrossID server dns name.

**`<REALM>`**: The Active Directory realm of the domain controller. note: To determine the realm on Windows 2016, right click on _My PC_ andd choose _Properties_, the _Domain_ is the realm name.

**`<user>`**: The user name created previously.

**`<file>`**: the file location to write the keytab to.

Example:

`ktpass /princ HTTP/crossidtest01.indexia.co@INDEXIA.CO /ptype KRB5_NT_PRINCIPAL /crypto AES128-SHA1 /mapuser crossid_sso /out c:\crossid.keytab`

![SPN And Keytab](/sshots/ad_auth/s4_spn_and_keytab.png)


If this error occurs:

```bash
DsCrackNames returned 0x2 in the name entry for crossid_sso.
ktpass:failed getting target domain for specified user.
```

try specifying the user in the format of: `<DOMAIN>\<user>` where domain is the domain name, for example:

`/mapuser INDEXIA\crossid_sso`


## Troubleshooting

### No matching key found

Error: `matching key not found in keytab. Looking for [HTTP server.domain.corp] realm: DOMAIN.CORP kvno: 5 etype: 18`
Problem: There is no matching key in the provided keytab
Troubleshooting: List the keys in keytab

```
ktutil
ktutil: read_kt <path/to/keytab/crossid.keytab>
ktutil: l -e
slot KVNO Principal
---- ---- ---------------------------------
  1     4      HTTP/server.domain.corp@DOMAIN.CORP (aes128-cts-hmac-sha1-96)
```

note: `ktutil` is provided by the `krb5-workstation` package.

- If looked KVNO is _5_ and keytab contains _4_ then keytab is too old.
- Make sure looked principal matches, otherwise keytab has a wrong SPN and should be re-generated.
- After updating keytab to the correct one, run in workstation: `klist purge` to force aquiring new ticket from KDC.
- Make sure looked etype (e.g., "18") corresponds to the encryption type of the key (e.g., "aes128-cts-hmac-sha1-96" is etype 17), see [assigned numbers](https://tools.ietf.org/html/rfc3961#section-8) for more info.
