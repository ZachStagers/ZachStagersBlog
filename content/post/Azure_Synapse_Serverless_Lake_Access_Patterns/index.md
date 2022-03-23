---
author: "Zach Stagers"
title: "Azure Synapse Serverless Lake Access Patterns"
date: "2022-03-23"
# description: ""
tags: 
    - "Azure"
    - "Synapse"
    - "Data Lake"
    - "Security"
image: banner_azure_synapse_serverless_lake_access_patterns.jpg
draft: "false"
---

I've been working with a client to build a large Data Lakehouse platform predominantly using Databricks as a processing engine and Synapse Serverless to serve the data out in a couple different ways. 

This was the first time I'd properly used Synapse Serverless in anger, and to say I found the documentation around lake access confusing would be an understatement! Everything I needed to understand was spread over a few different articles, which I've linked to at the bottom of this post.

The crux of it is that there's several ways to configure access to your lake data from Synapse Serverless, and in this blog I'd like to step through the ways we've configured it, and what those configurations are good for. This post does not speak to network security, although I have provided a link to the Azure Synapse Network Security white paper at the end.

### Active Directory (AD) Passthrough (a.k.a User Identity)
This is the default method of accessing data. If you simply open up your Synapse workspace and query your lake data using OpenRowset queries without creating and specifying a data source, your credentials will be passed down to the data lake layer to be authenticated.

This means that in order for authentication to succeed, you need to have configured either RBAC or ACL's for your user account;

* RBAC (Role Based Access Control) - Course grain access control. Whatever RBAC role is assigned will apply to all files and folders in the lake. The role applied to permit access via Serverless must be one of the 'Storage Blob Data ***' roles.
* ACL (Access Control Lists) - Fine grain access control. Grants access to specific files and folders. For a deeper look at ACL's, see a previous blog of mine here: [Azure Data Lake ACL Introduction](https://www.zachstagers.co.uk/p/azure-data-lake-acl-introduction/).

This option is great in two different ways, depending on which security protocol you use:
* For allowing analysts free reign into the lake (RBAC).
* To have meticulous control over what people have access to (ACL).

### Synapse Managed Service Identity (MSI) - Database Scoped
If you don't want to configure ACL's or RBAC for your users, you can grant access via the Synapse managed service identity (MSI) using a database scoped credential. Managed identities are essentially a credential that is created alongside the resource which can be treated like any other active directory account - they typically have the same name as the resource they belong to. The MSI will still need to be granted lake level access (i.e. via RBAC or ACL, per the above section).

To make use of the MSI, some set up is required. As we're creating a database scoped, you'll need to create a serverless database, then from within that database you need to create a master key to protect our credential.

Next we create a database scoped credential specifying that the credential is based on the managed service identity.

The final step is to create an external data source. This specifies our lake location via the data lake storage end point URL, and the credential to use to connect to that end point (in this case this will be our MSI credential). The location specified is the root of where access will be granted, meaning users accessing via this data source will have access to everything beneath the location specified.

Putting all of this together, we end up with this script to enable lake access:

```
CREATE DATABASE SomeDatabase

USE SomeDatabase

CREATE MASTER KEY ENCRYPTION BY PASSWORD='<SuperStrongPassw0rd>'

CREATE DATABASE SCOPED CREDENTIAL ManagedIdentityCredential
WITH IDENTITY = 'MANAGED SERVICE IDENTITY'

CREATE EXTERNAL DATA SOURCE DataLakeContainer
WITH (
      LOCATION = 'https://<storageAccount>.dfs.core.windows.net/container/',
      CREDENTIAL = ManagedIdentityCredential
     );

```

To use the data source, we adapt our queries to include a data source parameter in our OpenRowset. Notice now that the BULK parameter, which would by default include the entire lake path, now only needs the path down from the location specified in our data source.

```
SELECT
    TOP 100 *
FROM
    OPENROWSET(
        BULK '/folder/deltaTable',
        FORMAT = 'DELTA',
        DATA_SOURCE = 'DataLakeContainer'
    ) AS [result]
```

This option is again good in multiple ways:
* For allowing analysts free reign into the lake from a certain access point and below.
* For developing a standard set of views which expose data to PowerBI.

### Wrap Up
Using both of these methods together gives us a powerful blend of control and access. We're able to serve a standardised and trusted enterprise data model to PowerBI using the MSI method, and we're able to control access for analysts to splash around in the lake building their own models and metrics.

This isn't an exhaustive list of access methods - you could also use shared access signatures or service principals.

Check out some of the security documentation for yourself:

* [Serverless SQL pool in Azure Synapse Analytics](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql/on-demand-workspace-overview#security)
* [Control storage account access for serverless SQL pool in Azure Synapse Analytics](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql/develop-storage-files-storage-access-control?tabs=user-identity)
* [Self-help for serverless SQL pool](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql/resources-self-help-sql-on-demand#storage-access)
* [Azure Synapse Analytics security white paper: Network Security](https://github.com/MicrosoftDocs/azure-docs/blob/main/articles/synapse-analytics/guidance/security-white-paper-network-security.md)