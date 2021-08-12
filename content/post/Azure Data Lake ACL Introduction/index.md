---
author: "Zach Stagers"
title: "Azure Data Lake ACL Introduction"
date: "2021-08-11"
description: "Giving an introductory overview to securing an Azure Data Lake via ACL's and answering the question - \"What is the Mask ACL for?\""
tags: 
    - "Azure"
    - "Data Lake"
    - "Security"
draft: true
#image: manage_acl_mask_.jpg
---

### Introduction to ACL's

Access Control Lists (ACLs) offer low-level control of access to the folders within your Azure Data Lake, whilst Role-Based Access Control (RBAC) offers high-level control to the entire lake.

![RBAC vs ACL](datalake_rbac_acl.jpg)

The ACL permissisions on offer are:
* **Read** - Grants read access to files and folders (i.e. able to see the contents).
* **Write** - Grants write access to files and folders.
* **Execute** - Grants... execute. What this actually means is the user is able to navigate through the folder. A user must have execute assigned to the entire hierarchy above the folder they have read and/or write access to. Whilst in the portal you can assign this to files, it doesn't do anything.

As well as setting access explitly to a folder, defaults can be set at any level to inherit permissions to *newly created* sub folders and files, but note that these *do not* apply to folders and files which already exist.

Below is a visualisation of a simple folder structure with the minimum ACL's required at each level to read data from "File 1". Although this depicts the minimum permissions required to get to a single file, in reality you'd likely elevate the Read permission to "Sub Folder 1" as a default permission, therefore allowing it to inherit down to all files and folders listed underneath it.

![Data Lake ACL Example](datalake_acl_example.jpg)

There are a few things to be mindful of when testing with ACL's:
* Folder names are case sensitive.
* Testing via storage explorer won't work, you need to use something like Databricks or Power BI.
* You need to connect to the lake using the Data Lake Storage end point - `https://<lakename>.dfs.core.windows.net/<container>/<folders>`
* You must have a Read permission on the folder you're trying to connect to.

### So what's the Mask for?


![](manage_acl_mask.jpg)
