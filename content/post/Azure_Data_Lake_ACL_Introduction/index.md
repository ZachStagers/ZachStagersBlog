---
author: "Zach Stagers"
title: "Azure Data Lake ACL Introduction"
date: "2021-08-13"
description: "Giving an introductory overview to securing an Azure Data Lake via ACL's and answering the question - \"What is the Mask ACL for?\""
tags: 
    - "Azure"
    - "Data Lake"
    - "Security"
image: banner_azure_datalake_acl.jpg
---

### Introduction to ACL's

Access Control Lists (ACLs) offer low-level control of access to the folders within your Azure Data Lake, whilst Role-Based Access Control (RBAC) offers high-level control to the entire lake.

![RBAC vs ACL](datalake_rbac_acl.jpg)

The ACL permissions on offer are:
* **Read** - Grants read access to files and folders (i.e. able to see the contents).
* **Write** - Grants write access to files and folders.
* **Execute** - Grants... execute. What this actually means is the user is able to navigate through the folder. A user must have execute assigned to the entire hierarchy above the folder they have read and/or write access to. Whilst in the portal you can assign this to files, it doesn't do anything.

As well as setting access explicitly to a folder, defaults can be set at any level to inherit permissions to *newly created* sub folders and files, but note that these *do not* apply to folders and files which already exist.

Below is a visualisation of a simple folder structure with the minimum ACL's required at each level to read data from "File 1", whilst granting no access to "File 2". Although this depicts the minimum permissions required to get to a single file, in reality you'd likely elevate the Read permission to "Sub Folder 1" as a default permission, therefore allowing it to inherit down to all files and folders listed underneath it.

![Data Lake ACL Example](datalake_acl_example.jpg)

### Testing ACL's

There are a few things to be mindful of when testing with ACL's:
* Folder names are case sensitive.
* You need to connect to the lake using the Data Lake Storage end point - `https://<lakename>.dfs.core.windows.net/<container>/<folders>`
* If testing via storage explorer, connect to the lake using the 'ADLS Gen2 container or directory' option.

### The Mask ACL

When managing ACL's, there's a button to Add principal, which allows you to select and add a user or group to assign ACL permissions to. There's also a button to Add mask, and this defines an override of the effective permissions for named users and groups.

In the below screenshot, I've added a mask and removed all permissions. Next to my Zach Stagers named user with Execute permissions you see a warning symbol. If you hover over the warning, it'll say "The following access permissions are beyond the bounds of the mask: Execute". This is just highlighting to you that although Zach Stagers has Execute, it won't be effective because the Mask has disabled Execute permissions for all users.

![Manage ACL Screen with a mask applied](manage_acl_mask.jpg)

You might use this in scenarios where someone has put something sensitive or something they shouldn't in a folder in the lake and you want an admin to be able to go in and remove it whilst restricting access for everyone else. This is a much easier method than removing all of your assigned ACL's and then having to readd them.