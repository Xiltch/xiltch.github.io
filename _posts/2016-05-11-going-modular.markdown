---
layout: post
title: Going Modular
date: 2016-05-11 16:07:46.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- General
- Programming
tags:
- Thoughts
author: Jonathan Tweedle
permalink: "/2016/05/11/going-modular/"
---
So we have all played with Lego or at the least know about it. The idea is to provide a platform where anything can be built using the base building blocks. Now take that approach to a system solution...

This is where modular (or plugin based) systems come in. The idea being is to build out contained modules with the intention of connecting together in order to build out a full solution.

Some might ask why go through the hassle when you can hard code everything together avoiding the processing overhead cost?

Well much like 3D printing a specific toy, once built it is not so easy to break down and build something else like you can do with Lego.

The beauty with computers is the ability to clone files with a simple copy command. If you have something modular then you can copy-paste your modules, update config files and you have a new working system.

The key I have found to making this work is a solid host platform to build up from. Now it might seem tempting to include a bunch of nice features in that host but don't. The host must only focus on the core functionality.

Let's say your solution needs remote connectivity, If this was built directly into your host system and there was a bug in the code, once fixed you need to bring down the whole system to implement the fix. If this was modular then all that is needed is to reload that one module.

So the main host should focus on the following aspects to manage the system:

* Loading / Unloading modules
* Sending / Receiving payloads

The core behind the whole design is the payloads, a strongly typed message that the host defines and all plugins understand. It needs to identify what type of payload it is, where it is from and where it is going to. And then of course the contents of that payload.

Establish a basic set of payload types to allow low level routing. The payload type helps identify how either the host or a module should handle that payload without the need to process the contents. Basically the host is playing hot potato with those payloads and needs to avoid thread locks. It should avoid decoding the entire payload to ensure the system remains responsive to all modules. It should be unforgiving and never try to self heal a running module. The goal is let the modules running on separate threads do the heavy lifting.

Having a basic command type payload is key as well so that a module can directly control the host platform such as instructing the host to load a new module, change basic host settings, etc.

The next important part is the routing of payloads. Because the modules are doing the heavy lifting it makes sense to provide the ability for modules to communicate with each other through the host platform. The host needs to know who to send the received payload to while the receiver needs to know who sent it if a response is expected. Again like the payload type, you need to avoid decoding the whole payload.

Next is the ability to handle non targeted payloads or broadcasting, however there should be two varying types of broadcasting used. The first being where the payload is intentionally sent to all modules. This should be reserved only to your host and it's user limited to critical messages such as informing all plugins the system is being shut down in order to allow them to safely terminate without losing any data.

The second is more of a subscriber based model. The message is sent out to any plugin that had chosen to receive that style of payload. An example of this would be a logging system. All plugins can send a log payload to the host and then various logging plugins could subscribe in order to receive and consume those payloads. Some plugins would simply commit those logs to some form of storage medium where others could send emails based on triggers.

Ultimately the challenge is to ensure you cover all the foreseeable cases to mitigate any need to make changes to the host system. Remember that any change to the host could mean changes to all of the plugins as well so don't over complicate the host.

