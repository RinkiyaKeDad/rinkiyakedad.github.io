---
title: "Introduction To AWS IaaS"
read_time: true
date: 2021-10-02T00:00:00+05:30
tags:
  - aws
  - devops
---

This article is going to introduce you to the Infrastructure as a Service side of AWS. We'll start from the basics of how accounts in AWS work and cover some fundamentals in this article. Then in the next post, we're going to do something a bit more hands-on and see EC2 and S3 in action. 

## AWS Accounts? 

So let's talk a bit about AWS accounts first. If you just have a username and password for your account then you have what is called a root account. If you were sent a username, password, account id, and/or a special link to log in then you have an IAM user account. 

IAM stands for identity and access management. And this is how AWS controls who has access to your organization's account. Now, if you have an IAM user account, then your login screen will include a field at the top that says account ID or alias, and it should already be filled in for you. 

The AWS root account on the other hand comes with a whole bunch of special privileges. Most important of these being:

Ability to create IAM Users
Changing your support plan with AWS
Deleting the entire AWS account

These should give you an idea of why your org most probably provided you with an IAM user account instead of the root account :)

But if you do have a root account it's recommended you create an IAM user account for yourself and use that to follow through this article and the next one. You can do this by following the "Create a user from AWS Management Console" from [this](https://searchcloudcomputing.techtarget.com/tutorial/Step-by-step-guide-on-how-to-create-an-IAM-user-in-AWS) blog. Additionally, instead of directly creating a user, you can first create a user group and give this group a set of permissions and then create a user and assign it to the group. This would ensure you don't end up configuring permissions each time if you want to have multiple users. At the end of this, you should have an IAM user account, which would mean you have a login URL, username, and password. 

> Sometimes we need some keys to interact with AWS using our terminal. To get these, click on your username in the upper right toolbar, and in the menu, select "My Security Credentials". Under "Access Keys for CLI, SDK, and API access", click on the create access key button. From there copy both your "Access key ID" and "Secret access key". 


## Demystifying The Cloud

Let's delve into a bit of a history lesson here. Before IaaS services like AWS launched, to host your application you would need to self-host. Self-hosting meant building out a data center of your own or renting space in someone else's data center. This is what a data center looks like.

![data center](2021-10-02-1.jpeg)

A data center is mainly just a bunch of racks of servers. If you look at one of these servers you'll see that it's essentially just all the components you would find in your computer at home. It's just that these components are arranged in a different casing so that it slots easily into the rack to save space in the data center. So you would essentially buy one or more of these servers then you would either self-host in your room or take it to a colocated data center and slot it in a rack there. You could also rent one of these servers in a data center instead of buying your own. This had the added advantage of the data center people helping you if in case there was some hardware problem with the server. All this comes under on-premise hosting. 

With time virtualization started to appear in the server space market. A virtual server is just a software abstraction that allows a physical server to divide up its resources (CPU, RAM, hard disk, etc). This allowed a single physical server to appear to the outside world as five, or even ten separate servers. These virtual servers may even have a different OS from the physical server's OS. But how is all this relevant? 

Virtualization let data centers sell even more dedicated servers because now you can get maybe six virtual servers out of one physical server.

Using the on-premise approach had many problems like having staff to maintain and fix your hardware and the software running on it. Also, each server has a limit to the amount of traffic it can handle. So if over the festive season your e-commerce website got 20 times the normal traffic, the servers you had would overload and people won't be able to access your website. And for you to set up new servers it would take a couple of days at least. 

To solve all these problems AWS launched two services.

Elastic Compute Cloud (EC2) - these were like virtual servers you could rent from amazon. Many other companies were doing this too. But the keyword here is "Elastic". The EC2 instances came with a feature called Auto Scaling. This was the equivalent of the number of servers stretching automatically when traffic increases and then shrinking back down again when the traffic drops off. So when you got that 20x traffic EC2 would auto-scale your existing servers from the current capacity to the needed one for handling the load. Not only this but once the holidays were over and you had less traffic it would also scale down to the original number (or even lower) of servers you had. 

Simple Storage Service (S3) - think of this as Google Drive. It provided access to off-premise storage for your application.

Combine these two and you could now virtually build servers and storage solutions in the Cloud. This was the beginning of Infrastructure as a Service. 

## AWS Regions and Availability Zones 

Amazon has built multiple data centers near each other and it has linked them all together. These groups of data centers near each other form an availability zone. You won't know from which data center you're getting your resources from but you do get to choose your availability zone. AWS recommends designing your infra in a way that it can run simultaneously across 2 availability zones to avoid downtime as much as possible. For this, you'll see options to clone various AWS services into at least two availability zones. 

The availability zones themselves are clustered together by region. For example, all of the availability zones near North Virginia are called the North Virginia Region or us-east-1. The availability zones within North Virginia are labeled as us-east-1a, us-east-1b, and so on. 

When selecting a region the general guideline is to pick a region closest to most of your users and which supports the AWS services that you want to use because not all regions offer the exact same service.


Well, that was it for this post. Now that you have some idea of what AWS does and offers when it comes to IaaS, in the next post we'll be seeing the EC2 and S3 we talked about briefly here in action. Thanks for reading and if you have any questions or feedback please feel free to [reach out](https://arshsharma.com/) to me. 