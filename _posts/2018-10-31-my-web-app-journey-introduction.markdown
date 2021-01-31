---
layout: post
title: My Web App Journey - Introduction
date: 2018-10-31 07:56:58.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
tags: []
author: Jonathan Tweedle
permalink: "/2018/10/31/my-web-app-journey-introduction/"
---
I finally rebooted a project with the goal of building a web application to replace on old desktop application I put together a few years ago using C#.

It is a simple application, used to aggregate bank transactions across multiple accounts into a single data store. When importing new data it categorizes each transaction based on user defined rules. This culminates into analyzing your transactions across the various categories, allowing you to discover how much your spending on things like Take-Aways or on your Entertainment and so on.

I plan to reboot the project but using different a different technology stack to help me learn so this blog is going to document my progress through my journey in the hopes it might help others trying to learn certain parts of the stack as well.

The technology stack I plan to use is as follows:
- MongoDB to house the data
- Node.js for the API endpoint and hosting the UI
- Typescript as the core language
- Raspberry Pi to host the web app
- Angular to design the UI


It is possible the stack might evolve over time but for now this is the current plan.

Due to the nature of the project, I want to keep the web app local to my home network which is why I settled on node.js standing on top of the raspberry pi. This will allow the app to be used from any device and allow multiple users to use it. The decision to use the pi is the low cost hardware and because I wanted to practice my Linux skills.

### The Big Picture
The application is going to be composed of different layers. The approach being to design it as an n-tier application. The data layer being limited to looking after the data, the API layer to make the data accessible through web API calls, and finally the UI layer to allow the user to interact and make calls through the web API.

The [next chapter][next_chapter] will be the preparation of both the development and production environments used to develop the app.

[next_chapter]: {% post_url 2018-11-02-my-web-app-journey-preparation %}