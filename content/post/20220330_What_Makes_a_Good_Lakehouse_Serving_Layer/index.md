---
author: "Zach Stagers"
title: "What Makes a Good Lakehouse Serving Layer?"
date: "2022-03-30"
tags: 
    - "Azure"
    - "Data Serving"
    - "Data Lake"
    - "Lakehouse"
image: banner_what_makes_a_good_lakehouse_serving_layer.jpg
draft: "false"
---

This post is a bit of an exploratory one, collecting some thoughts about features which combine to make a good data lakehouse serving layer, and the things to consider when choosing a resource to serve your data.

I do not intend for this post to act as a guide for choosing the right serving layer, but instead it's an introduction to the subject and some things to watch out for as you identify which tool would be best for you.

### What is a serving layer?

Simply put, a serving layer is a method of presenting data to users. Or put differently, it's a tool through which users can access data.

But when you break that simple statement down, there's a lot to think about! What data is being presented back, and who is it being presented to? How do users want to connect to the serving layer (assuming they have a choice)? And when they've connected, how do we ensure they only see the data they're supposed to see? How much data is there? How many users are there? What timezones do they operate in?

Taking the answers to these questions, we can build a picture of our requirements, and therefore the features we need to think about and therefore what makes a serving layer good.

### Why do you need a serving layer?

A big selling point of the data lakehouse is the lakes ability to service a broad range of users, directly from the lake. But  all the lake does is store data - it has no compute ability with which to query that data.

Regardless of whether you have analysts exploring the lake or you have a centrally-controlled and developed enterprise model, you need a compute resource to query that lake data.

### What features do we look for in a serving area?

**1. Concurrency**

If we know we have 100's or even 1,000's of users who'll be querying our data model every day, it's no use copying our data into a Synapse Dedicated pool. Not only would that be an expensive option, but dedicated pool's maximum concurrency limit of 128 would quickly cause delays and performance issues. It would also defeat the purpose of the lakehouse paradigm by duplicating your data to somewhere outside of the lake.

We need to ensure the tool can handle the number of users and queries, remembering that each visual on a dashboard equates to a query, this can add up fast.

**2. Security**

Row Level Security (RLS) has long been method for enhancing data security at the serving layer, and is often an important feature to look out for in the majority of projects.

Other things to consider here is how well it integrates with your identity management system, whether that be Azure Active Directory (AAD) or something like Okta.

**3. Performance**

Performance is one of those things that can go completely unnoticed when something is fast, but can be the bane of everyone's life when it's slow! 

Some things to consider in this area are whether the first users to query each day will have to wait for a resource or cluster to start, or in a serverless scenario, how long does it generally take for a query to even *begin* running.

On the other hand, if we're importing data into a PowerBI model, how long does that process take? This is especially important if we want to refresh our models throughout the day. 

**4. Integration**

There's two sides to the serving layer to consider here. The integration between the lake and the serving layer, and the integration between the serving layer and user reporting tools like PowerBI.

On the lake to serving layer integration, if we're using a direct-query model, it absolutely must understand how to read a parquet file or delta table to be a suitable option for serving a data lakehouse. Without this, querying any datasets on the larger side would likely be unacceptably slow for users.

Whereas on the other side, does it play nicely with the reporting tools users will be connecting with such as PowerBI, Tableau, Looker, ...or Excel.

**5. Cost**

OK, cost isn't really a *feature*, but it's almost always one of the biggest factors in choosing any component of a data platform.

Things to consider when you're on a tight budget is whether the resource needs a cluster up and running, and if so what the uptime scheduling for that looks like and how big the cluster needs to be to support the number of users you're expecting.

### What options are there?

Gone are the days of Analysis Services' dominance in the serving arena! There are a host of tools available which offer lake based serving capabilities.

* Azure Synapse
* Databricks
* Azure Analysis Services
* PowerBI (Import Mode Only)

These are obviously very Azure / Microsoft focused, which is the space I tend to operate within, but of course there are plenty of other vendors which can read and serve lake data too.

### Conclusion

As I've written these thoughts and considerations out, it's dawned on me just how complex and nuanced a topic this is!

In a future post I'd like to explore the serving options further, their strengths and weaknesses in the areas I've mentioned in this post, and the circumstances under which each may be the most appropriate choice.

I'd be interested to hear other peoples thoughts on this subject.