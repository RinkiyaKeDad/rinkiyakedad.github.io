---
title: "Platform Engineering and Developer Experience: Two Sides of the Same Coin"
read_time: true
date: 2023-09-23T00:00:00+05:30
tags:
  - platform engineering
  - devex
---
Around a year ago, the term "developer experience" was all the rage. It was on everyone's lips, with discussions about its immense importance. However, the current focus seems to have shifted towards "Platform Engineering" - a term that, in my personal opinion, has become quite convoluted. Nevertheless, **I feel** both concepts essentially revolve around the same idea. Platform Engineering is merely a means to achieve a better developer experience. But what exactly does that mean? Let me explain in more detail over the course of this article.

![photo of a coin](coin.jpg)
Photo by [Mateusz Dach](https://www.pexels.com/photo/silver-coin-selective-focus-photography-1431158/)

## Time for a History Lesson

Before the advent of microservices and containers, software developers held the primary responsibility of writing code for the applications to be used by end users. Their focus was solely on this task. Once the application was complete, it would be handed over to either the SysAdmins or Ops people (depending on your preferred dialect of corporate jargon). This clear separation of concerns meant there were distinct individuals dedicated to writing application code and others entrusted with building and deploying the applications.

Then Docker emerged, popularizing containers, which in turn sparked the rise of DevOps within organizations. Software developers were given additional responsibilities of creating Dockerfiles and managing application builds, based on the notion that they were already responsible for writing the code so why not build it as well. This idea gradually extended to other areas, with developers being expected to handle other parts of the software delivery lifecycle. This gave birth to the role of DevOps engineer, individuals who possessed knowledge of both the application stack and the production side of things such as build and deployment pipelines. As more developers got involved in infrastructure and deployment side of things, the lines started to blur. The landscape became even more complex with the proliferation of tools, leading to an overwhelming cognitive load for these so-called DevOps engineers. This is where we currently were when we started hearing about Platform Engineering.

## Platform Engineering - Realizing That We Messed Up

To be honest, the concept of "DevOps" has never quite resonated with me. It seems impractical to expect a single engineer to excel in both development and operations. These are distinct fields, each requiring a profound understanding of its own domain. Unfortunately, this has led to a situation where cognitive overload burdens engineers, resulting in a subpar developer experience. Experienced and senior application developers have taken on the role of DevOps engineers, juggling their new "dev" and "ops" responsibilities. As a consequence, they have less time to focus on what they are truly skilled at - writing quality code to build products. 

Simultaneously, they also face the challenge of meeting the requirements of traditional software developers who rely on them for development environments, staging environments, and other resources to support their development and testing processes. The approach of coding microservices-based containerized applications in the same way as we did monoliths, proved to be impractical. Creating a local environment that mimics production became increasingly unfeasible, leaving developers with two options: either learn cloud and Kubernetes to replicate production environments to test their applications or depend on "DevOps" engineers for assistance.

Platform Engineering promises to swoop in to save the day, resolving all the chaos. 
It involves equipping different teams in your organization with the necessary tools and establishing efficient workflows, enabling each of them to be self-reliant. The responsibility of platform engineers is to implement an Internal Developer Platform (IDP), designed to serve as the company's internal product, with developers as its primary users. But how does this actually untangle the mess we were discussing earlier?
Once an IDP has been implemented and teams become self-sufficient, everyone can focus on their strengths instead of trying to handle everything on their own. This prevents feeling overwhelmed and frustrated in the end. The platform teams serve as a bridge between different teams, facilitating the exposure of each team's services to one another, allowing everyone to excel in their respective roles. Developers can concentrate on writing code without the burden of setting up environments every time. It becomes the responsibility of the platform teams to provide self-service environments for development and testing for these developers. Similarly, some DevOps engineers may find joy in their new Ops roles and prefer to focus solely on deployment. This is ideal because now they can fully dedicate themselves to that task, as the platform team takes on the responsibility of handling infrastructure requests from developers. In summary, platform teams implement an internal platform that allows each team to focus on their expertise, while the platform team acts as the glue that connects all these different teams together.

## So What's This Got to Do with Developer Experience?

One of the key goals in platform engineering is achieving separation of concerns. Once this is achieved, the developer experience is significantly improved as a by-product. Whenever one team becomes dependent on another, delays occur, resulting in a negative developer experience. Similarly, when one team takes on responsibilities that should ideally be handled by another team, it adds to the cognitive load on developers, leading to a poor developer experience as well.

Platform engineering aims to address these precise challenges by providing self-service and automated solutions for various teams to utilize. This approach ensures that teams are not reliant on each other and can focus on their primary responsibilities. The platform team takes on the responsibility of enabling seamless discovery of one team's services by another, facilitated by an Internal Developer Platform. This is what folks in the industry mean when they talk about "golden paths". Golden paths are ways that the platform team sets up to help each team achieve their goals without going through a tedious workflow or relying on others. These golden paths are what make the developer experience better for everyone in the organization.

## Conclusion

In a sense Platform engineering brings us back to the traditional model of separating concerns and allowing everyone to focus on what they're best at. It provides a solution to the chaos that emerged from trying to merge development and operations. By implementing an Internal Developer Platform, platform engineers enable teams to become self-sufficient, leading to a better developer experience and ultimately resulting in higher-quality products being delivered. Embracing platform engineering promises to be a way to unravel the mess we created with DevOps and bring back a more straightforward and efficient way of working. So the next time you hear about platform engineering, remember that it's not just another buzzword but a concept that has the potential of transforming your organization for the better!

As always if you liked this blog, consider following [me](https://arshsharma.com/) on [Twitter](https://twitter.com/RinkiyaKeDad) or [Dev.to](https://dev.to/rinkiyakedad) to stay updated whenever I share similar content :)
