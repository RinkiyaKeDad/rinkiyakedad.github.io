---
title: "So What Even Is This Cloud Computing?"
read_time: true
date: 2021-09-24T00:00:00+05:30
tags:
  - devops
---

Cloud computing is what everybody seems to be talking about these days? But what does it even mean? In this short article, I'll go over what it is and the different types of cloud deployment and delivery models.

## What is Cloud Computing?

Earlier to host applications you would have to rent or build data centers where you would store the hardware which would run your servers. 

Cloud computing simply refers to using computing resources on demand without directly managing them. So instead of building data centers and buying hardware to run your apps you simply rent compute power and use that to suit your needs. Who do you rent it from? Let's get to that later in the [cloud delivery models](#cloud-delivery-models) section. 

The biggest advantage of cloud computing is that instead of spending the cost of running an entire data center you only pay for the power you need. This also includes scaling up and down based on your needs. All you need to do is connect with your cloud provider and from their provided console change the resources you're using. 

Behind the scenes, these cloud providers are still running and maintaining data centers of their own but that's none of your headache now. You just request the resources you need and you'll be provided with them from a shared pool of resources that they maintain. Once you're done using them those resources go back to this shared pool. 

Now that we have an idea of what cloud computing is let's see what different types of cloud deployments are available.

## Types of Cloud Deployments

1. Private: Like the name suggests Private cloud means that the hardware on which your cloud runs is owned by you or your organization.

2. Public: Public cloud means that the hardware isn't owned by you and the organization which owns this hardware provides you access to the computing power you pay them for.

3. Community: It is kind of like Public cloud, but it differs in the fact that access to the computing power is only allowed to a particular group of organizations or people who belong to a particular community.

4. Multicloud: This means you're using resources from two or more public clouds or two or more private clouds.

5. Hybrid: Hybrid clouds allow you to use resources offered by both private and public clouds.  

There isn't any one best approach as such. Private clouds offer more security since the resources are provisioned for use by a single organization. The computing resources are not **multitenant**, meaning that systems are not shared while provisioned. 

Public clouds provide resources that are generally multitenant, meaning they are shared between different users. When one user is done using the resources they are added back to the shared pool and are available for other users. These are much more cost-effective and easier to manage since you don't have to maintain any hardware. Security these days is almost as close to that provided by private clouds.

Multicloud uses the cloud platforms of multiple cloud providers. So it's when you use a mix of [AWS](https://aws.amazon.com/), [Azure](https://azure.microsoft.com/), [GCP](https://cloud.google.com/), or other cloud providers. The reason you'd want to do this is that some cloud providers might be better than others at certain types of services. It also ensures that you aren't solely dependent on a single cloud provider. However, this does mean that setting them up ends up being a bit more complicated than using a single cloud provider. 

## Cloud Delivery Models

### Software as a Service (SaaS)
In the SaaS model, a company hosts the application you need and provides it for you. The SaaS provider may internally use any model for the compute resources they need in order to provide you with their software. An example to make this clear is Google Workspace. Google Workspace features many SaaS productivity applications like Gmail, Calendar, Docs, etc. They make lives easier compared to the non-cloud alternatives since users now don't have to buy the hardware and software to support an on-premise business application. 

### Infrastructure as a Service (IaaS)
This model refers to using the compute resources traditionally found in an organization's own data center over the internet. What this basically means is that someone will host the hardware for you and provide you access to the computing power over the internet. [AWS](https://aws.amazon.com/), [GCP](https://cloud.google.com/), and [Azure](https://azure.microsoft.com/) are examples of IaaS clouds. The main benefit they provide is that you don't need to buy the hardware and you pay only for the resources that you use. 

### Platform as a Service (PaaS)
Under the PaaS model, a company provides the tools needed for application development, testing, and deployment to users over the internet. These days most PaaS solutions support IaaS cloud providers. For example, [Amazon Web Services Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) is a PaaS cloud that runs inside an IaaS cloud, that is, [AWS](https://aws.amazon.com/). PaaS offerings increase developer productivity by providing abstraction from the complexities of development environments. 

So this was it for this article and I hope you now have more clarity on what cloud computing means and the various cloud computing models available. If you've any questions or feedback, please feel free to reach out to me on [Twitter](https://twitter.com/RinkiyaKeDad).
