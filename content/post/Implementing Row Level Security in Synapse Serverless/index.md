---
author: "Zach Stagers"
title: "Implementing Row Level Security in Synapse Serverless"
date: "2021-08-23"
description: "How to implement bi-directional row level security (RLS) in Synapse Serverless SQL Pools, to act as a serving layer over an Azure Data Lakehouse."
tags: 
    - "Azure"
    - "Synapse"
    - "Security"
    - "Data Lake"
draft: true
image: banner_azure_datalake_acl.jpg
---

### Introduction

You've built your brand-spanking-new Azure Data Lakehouse, you've carefully curated your dimensional facts and dimensions, and you're ready to serve your data. You want analysts to be able to connect up to the model using a plethora of tools such as Power BI, SQL, Tableau, Python... but no matter what tool being used, you want all users to be subject to the same row level security (RLS) rules.

There's no native functionality to implement RLS directly within a data lake, so something needs to be put over the top of the lake to implement it. This blog covers implementing RLS over a data lake in Azure Synapse Serverless SQL Pools, and how to make it bi-directional if required.

### Example data in the lake

This is what our example data model looks like in the lake, and what it contains

### Querying the lake using Synapse Serverless SQL Pools

Linked lake, right click select *...

### Implementing row level security (RLS) via Synapse Serverless SQL Pools

Creating views...

Function to implement in all views...

