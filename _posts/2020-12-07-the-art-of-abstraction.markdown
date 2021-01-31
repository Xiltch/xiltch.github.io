---
layout: post
title: The art of abstraction.
date: 2020-12-07 12:33:12.000000000 +00:00
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
- Technology
- Thoughts
author: Jonathan Tweedle
permalink: "/2020/12/07/the-art-of-abstraction/"
---
Imagine for a moment that you have your own business and as the only employee you are responsible for everything. This involves the sourcing of raw or input value into the business. Processing of the input in order to generate some kind of value, marketing, etc... I think you get the point. Imagine the business takes off resulting in more customers. In order for the business to continue, it needs to scale to the demand. At some point it will become impossible to manage solely and you will be forced to delegate responsibility.

This is where the "Art" comes in to play because it really is something of an interpretation.  There exists countless books that cover many different management styles with the hopes of maximizing the efficiency. Techniques which span the full spectrum from basically nothing to obsessive micro management. 

The software development space is no different, with no single correct answer either. Over time as the team expands it will ultimately evolve, tweaking the development process. 

Part of that evolution is having to maintain existing solutions. You can either keep patching it up but let's say that solution was based on a stack from 1960 ... Eventually you will reach a ceiling where it becomes impossible. Lack of hardware support, lack of software support, and the lack of people who are skilled in that area. The other option is to rebuild using newer technologies and skills, with some degree of future proofing, but even that will eventually become outdated.

Thus one needs to abstract responsibility of the design. Just like in the business example, your sourcing and invoicing can be completely isolated. They do not need to understand how either one functions. The sourcing department simply informs the invoicing department of a new invoice from a supplier, requesting it be paid. They don't need to know how it gets paid, only that it is paid so they can keep going to the supplier.

In software development this is referred to as a "Black box". It is a term used for testing code, internal solution design, but also expands beyond to the platform and architecture design. When designing a large scale platform, if you had to worry about the lower level code design, you would never be able to finish planning. Even if you you did finish, it would take so long that the landscape has changed, forcing you go back and plan everything again.

Personally my goal is to isolate as much functionality where possible, breaking the code design up into the smallest blocks of logic possible. This sounds counter productive, but will add flexibility to swap out pieces of the solution bit by bit. Let's say you want to take a monolithic application and convert it into something that is more micro service based? Start by switching over one or two small modules into a micro service with a service layer to interact with the new micro service. At some point your monolith is reduced to a collection of service layers that interact with different micro services. At which point you can design a new application that targets a new technology without having to redesign a substantial portion of the code base.

Obviously that sounds a lot easier than in practice because you might have to redesign whole layers due to scaling issues, and how problem space will the influence choices which can impose restrictions on the solution. Hopefully through some trial and error you will make it through the gauntlet ready to face the next battle of evolving code.
