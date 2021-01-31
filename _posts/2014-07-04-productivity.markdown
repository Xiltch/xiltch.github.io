---
layout: post
title: Productivity
date: 2014-07-04 17:52:30.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- General
- Programming
tags:
- design
- patterns
- refraction
- Thoughts
author: Jonathan Tweedle
permalink: "/2014/07/04/productivity/"
---
As a developer this is most likely an area you will face at some point in your professional career.

In the workplace you have to face deadlines and meeting those deadlines is a challenge closely related to how productive you are as a programmer.

I always face the battle of deciding exactly how to solve a problem. Sure you could always resort to a brute force approach but at what cost?

Over the years the manner in how I go about solving problems has in some ways drastically changed, attributing to my experience as a software developer.

A simple approach is the following three step process:

1. Make it work.
2. Make it work better.
3. Make it work faster.

In order to make this approach easy to incorporate you generally need to follow some design pattern. Now there are many patterns to choose from and I would be surprised if you could not find a person to say why any such pattern is not worth using. The bottom line is that if you don't use any such pattern your code will be very hard to maintain.

Yes you could simply write those lengthy single function applications, where the entire code is contained in a single submit click method. As your solution evolves into a 14 page thesis, you start to wonder if the rush to get the solution finished was worth it? Unless your IQ is above 200 and you inherited clairvoyance from your distant aunt Cleo, gifting you with ability to write perfect code and see the matrix in real time like the one, you will sooner or later need to maintain it.

I favor the object orientated approach by segmenting the solution into smaller chunks. Debugging becomes easier even if you use debug execution with break points. The main goal being to create each chunk with a specific behavior, given input and output. This means that you can now modify the inner workings of each chunk that hopefully does not break your whole solution. This could possibly be improving the way your solution sorts a list of items and removes duplicates. How this is fine becomes irrelevant and only the result is critical.

This process of abstracting and refactoring my solutions I have found helped to improve my solutions over time. Over time my experience has granted me the wisdom to decide on an appropriate design.

This has been in the form of test driven development, custom attributes, reflection, generic types and interfaces. Where possible I try to maximize code reuse and lately my goal has been to collate logic together as opposed to have multiple methods dotted throughout my class design.

One solution I have slowly created is a plugin based system that allows each plugin to send messages to each other using XML messages. This allows one plugin to trigger an action in another plugin providing a form of late binding, it also means the plugins don't require implicit reference to another plugins method's or properties or even function. Part of this solution is a command handling design that matches regular based expressions and triggers specific methods based on the matching expression. I relied on a list of the expressions paired with delegates to relevant method calls. The problem here was the definition of the list and the actual methods were separated. I later improved on this design by implementing custom attributes that could be defined right before the method implementation keeping both together. I added a constructor for my custom list that would accept a reference to the parent plugin that would use reflection to search for any methods with the custom attributes and add them to the list dynamically. The reason for this change was to improve the design to help other developers support my solution in the future.

In the end I have realized that yes there is that perfect code I am yet to create but until then I need to not lose sight of what I am working at that moment. I try to make sure I structure my design that will allow to refactor my code if it even needs to be improved.

