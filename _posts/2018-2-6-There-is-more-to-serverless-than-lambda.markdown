---
layout: post
title: "There is more to serverless than lambda"
date: 2018-2-6 16:00:00
comments: true
categories: serverless AWS FAAS StepFunctions lambda
---

# There is more to serverless than Lambda.

In case you've been living under a rock let give you a brief introduction.  Amazon and other cloud providers have this thing called FAAS (Functions As A Service).  FASS is the next step in cloud infrastructure that allows developer to write functions and deploy them to be executed in response to an event.  Many organizations are starting to use FAAS because of two very important characteristics of these systems.  

1. You don't pay for idle time.  If you code is not being exectuted you aren't being charged.
1. You can scale from 1 req/s to 1000 req/s or more with absolutely zero effort or planning.

Just to manage your expectations of this article, I'm going to focus this effort on AWS but the knowledge is applicable to other providers as well.

# So that's it just deploy functions and you are done right?

Not so fast, creating a function and executing it is just one piece of the puzzle.  There are a lot more questions one might ask before one decides to use FAAS in production.  The rest of this article will be an attempt at answering the following questions.

1. How do I manage multiple environments? (Production, Stage, Dev)
1. How do I develop locally?
1. How do I co-ordinate multiple functions as part of one process?
1. What is the proper level of granularity for my functions? (how much work should one function do)
1. What language should I use?

There are other questions that you may have, like where should I persist data or how should I manage secrets.  Those are great questions that I may take time to answer in the future but I think the list we have so far is enough to get us started.


### How do I manage multiple environments?

When you make the step into serverless you are making a secondary decision whether you know it or not and that decision is that you are also going to write infrastructure as code. Don't panic, this is a good thing. What this means is that if you want to stand a chance of creating a production system with multiple stakeholders you are not going to get away with clicking through the AWS console (the console is for demos and debugging).  So you are going to have to get familiar with some Dev Ops tools.

There are a couple of competing choices and much of the decision you will make is likely to be based on the following.

1. What is your organization currently using?
1. Multi-Cloud or Uni-Cloud?
1. Code or Config?

If your organization is already using a devops tool that integrates well with cloud providers then you will likely go that route and there is no reason to read this section.  

A popular tool these days is [Terraform](https://www.terraform.io/).  Terraform provides a domain specific language for managing cloud resources across a number of providers.  Furthermore Terraform has some language features that set it apart for config based systems (like cloudformation).  You probably can't go wrong with Terraform if given the choice. 

Another option is to use the [serverless framework\(https://serverless.com/).  This is a great option if you are only focused on serverless and don't have intentions of introducing more traditional services running in say docker containers or dare I say on premise.  Serverless is great if you are on a greenfield project that only requires serverless.

If you are using AWS and only ever plan to use AWS I would recommend using Cloudformation with the SAM transformation.  This allows you to get some shortcuts for common serverless use cases while still using Cloudformation to manage your deploys.  Cloudformation integrates very well with many of the code management services on AWS which can reduce the amount of custom code deployment management tooling you need to write yourself.

Once you have decided on a framework, managing multiple environments is really just a matter of namespacing functions and supporting services such that one can easily understand what environment each component belongs too.  For example if you create a Step Function or IAM Role and allow the default naming to happen you won't know if those things belong to Dev or Production when browsing around the AWS console which can pose problems for the un-initiated.

The second thing you will want to pay attention too when managing multiple environments is your infrastructure Outputs.  Let's say you create a new function and a new Api Gateway enpoint.  Well you'll want to make it so that you can very quickly query your infrastructure to get the latest values for these endpoints and use them in dependent projects. 

For example, let's say you want to send a notification via SNS (Amazon's Simple Notification Service) furthermore let's say you have two environments (Production, Staging).  How would your application know if you were generating the notification for Staging or Production if you didn't generate a config file for it to use?  The answer is that the calling application wouldn't unless you had explicitly told your infrastructure provisioning tool to output that information. 








