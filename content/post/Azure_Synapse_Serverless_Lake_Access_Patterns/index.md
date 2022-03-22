---
author: "Zach Stagers"
title: "Azure Synapse Serverless Lake Access Patterns"
date: "2022-03-21"
# description: "Giving an introductory overview to securing an Azure Data Lake via ACL's and answering the question - \"What is the Mask ACL for?\""
tags: 
    - "Azure"
    - "Synapse"
    - "Data Lake"
    - "Security"
image: banner_azure_datalake_acl.jpg
draft: "true"
---

I've been working with a client to build a large Data Lakehouse platform predominently using Databricks to process data, but we're using Synapse Serverless views to serve the data out in a couple different ways. 

This was the first time I'd properly used Synapse Serverless in anger, and to say I found the documentation around lake access confusing would be an understatement!

There are several ways to configure access to your data from Synapse Serverless, and in this blog I'd like to step through some of the ways we've configured it, and what those configurations are good for.

### Active Directory (AD) Passthrough
This is the default method of accessing data. If you simply open up your Synapse workspace and query your lake data using OpenRowset queries without creating and specifying a data source, your credentials will be passed down to the data lake layer to be authenticated.

This means that in order for authentication to succeed, you need to have configured either RBAC or ACL's for your user account;

* RBAC (Role Based Access Control) - Course grain access control. Whatever RBAC role is assigned will apply to all files and folders in the lake.
* ACL (Access Control Lists) - Fine grain access control. Grants access to specific files and folders.

For a deeper look at ACL's, see a previous blog of mine here: [Azure Data Lake ACL Introduction](https://www.zachstagers.co.uk/p/azure-data-lake-acl-introduction/).

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

Users can now query their little hearts out, without having any direct access to the lake themselves.

To further abstract access away, a set of views can be built on top of the data source, and users can be granted access to those views only. 
