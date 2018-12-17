---
title: Create OAuth application
linktitle: Create OAuth application
description: Explains how to register application in Azure AD.
date: 2018-12-02
publishdate: 2018-12-02
lastmod: 2018-12-02
categories: [integration, azure, cloud]
layout: single
weight: 20
sections_weight: 20
draft: false
toc: true
---

Authentication and access to Azure AD resources (such graph API) require setting up [service to service authentication flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow), this guide walk you through app registration.

Out of this guide we need the folliwing parameters:

- Client ID
- Client Secret
- Directory ID

## Create App

- Login to [Azure Portal](https://portal.azure.com)
- Click on _Azure Active Directory_
- Click on _New application registration_

![How to create app #1](/sshots/azure_ad_reg_app/azure-ad-reg-app-1.png)

Within the _Create_ app form, fill in the following properties:

- Name: _CrossID_
- Application Type: _Web app / API_
- Sign-on URL: `https://<tenant_id>.crossid.io`  (e.g., http://indexia.crossid.io)

![How to create app #2](/sshots/azure_ad_reg_app/azure-ad-reg-app-2.png)


## Add required permissions

Once application is created, add required permissions:

Click _Settings_, then _Required permissions_, _Add_, _Select API_

![Add permissions #1](/sshots/azure_ad_reg_app/azure-ad-reg-app-3.png)

Select _Microsoft Graph_ from the selection and press _Select_

![Add permissions #2](/sshots/azure_ad_reg_app/azure-ad-reg-app-4.png)


Select _Read directory data_ from the access list and press _Select_, then press _Done_

![Add permissions #3](/sshots/azure_ad_reg_app/azure-ad-reg-app-5.png)

Optionally select _Read and write directory data_ if write access is neccessary.


Click _Grant Permissions_ and then _Yes_ to grant the selected permissions to the app.

![Add permissions #4](/sshots/azure_ad_reg_app/azure-ad-reg-app-6.png)


## Add key

- Add a key by selecting _Keys_ from the _Settings_ menu
- Under passwords table add some key description (e.g., _primary_), select duration and press _Save_

![Add keys #1](/sshots/azure_ad_reg_app/azure-ad-reg-app-7.png)


Copy the generated key _Value_ (client secret) and _Application ID_ (client ID) to a safe location.

![Add keys #1](/sshots/azure_ad_reg_app/azure-ad-reg-app-8.png)

Both client ID and secret are required for the integration to work properly.


## Finding your directory ID

In portal navigate to _Azure Active Directory_ -> _Properties_ and copy the _Directory ID_ property.

![Azure AD Directory ID](/sshots/azure_ad_reg_app/azure-ad-directoryid.png)


## References

- [Adding app in Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-v1-add-azure-ad-app)
- [Azure OAuth client credentials flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow)
