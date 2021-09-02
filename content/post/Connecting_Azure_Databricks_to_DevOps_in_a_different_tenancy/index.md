---
author: "Zach Stagers"
title: "Connect Azure Databricks to a DevOps repo in a different tenancy"
date: "2021-09-01"
description: "How to connect Azure Databricks to a DevOps repository held within a separate tenancy using a Personal Access Token."
tags: 
    - "Azure"
    - "Databricks"
    - "DevOps"
image: banner_databricks_devops_access_token.jpg
---

If you're working with a large multi-tenancy organisation it's possible the subscription your Databricks resource sits in is a different tenancy to the Azure DevOps hosting your repositories. This blog explains how to connect Databricks to a DevOps repository in that scenario.

When trying to connect to DevOps in a seperate tenancy, you'll receive the message `Unable to parse credentials from Azure Active Directory account. Ensure Azure Devops account is connected to AAD.` if you haven't configured a Personal Access Token (PAT).

1. Go to dev.azure.com and login to the DevOps organisation containing the repository you're trying to connect Databricks to.

2. Click the User Settings icon in the top right and go to Personal Access Tokens.

3. Click + New Token

4. Fill in the Create a new personal access token form:
    * Give the token a sensible name, such as 'Databricks Repo Token'
    * Select the appropriate organisation
    * Set the expiration as required
    * The scope required 'Full access'
    * Press Create

5. Copy the access token displayed.

6. Go to your Databricks workspace.

7. Click the workspace name in the top right and choose User Settings

8. Go to the Git Intergration tab at the top of the page.

9. Change the drop down to Azure DevOps Services (personal access token)

10. Populate the Git provider username or email address with the email address you use to log in to the DevOps organisation.

11. Paste the token copied in step 5 into the Token box and press save.

With that configured, you can now go back to Repos, select Add Repo, and clone the remote Git repo. 

