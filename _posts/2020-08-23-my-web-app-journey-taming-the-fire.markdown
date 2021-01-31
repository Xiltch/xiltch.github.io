---
layout: post
title: My Web App Journey - Taming the fire
date: 2020-08-23 15:31:41.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
tags:
- My Web App Journey
author: Jonathan Tweedle
permalink: "/2020/08/23/my-web-app-journey-taming-the-fire/"
---
In my [previous post][previous_post] I was planning to get back into my project but then the whole world turned into flames. Work remained constant but spare time was soon occupied with games and again my ambitions receded once more.

But like a phoenix I once more will rise from the ashes ... reviving the ol Raspberry Pi and once more march on.

## Baking the Pi

Originally I was planning on using Node.js to stand up the server and while getting that going was fun, at heart I am a C# developer. With the .NET Core 3.1 release the appeal of running .NET Core on my PI was too good to pass up.

I stumbled across a [talk/demonstration][youtube_pete] by Pete Gallagher showing how to run .NET Core 3.1 on a Pi. This was the spark to rekindle my fire, followed the [guide from his post][blog_pete] which he made even simpler with a handy script.

## A stretch of the imagination

With the script completed I checked that dotnet tool version 

```
dotnet --info
```

Everything looked promising, I showed that .NET Core SDK 3.1.302 was installed along with a bunch of other stuff so in theory it should work now. I created a new directory aptly named 'helloworld' and made a new project.

```
dotnet new console
```

It worked! I now had a very basic project and all that was left was to fire it up.

```
dotnet run
```

Alas it was too good to be true. It did not work. I tried to build then run ... no joy. It just said i needed to install the binary for the ARM processor. So i thought maybe it was because I was running Raspbian stretch (OS Version 9) instead of using buster (OS Version 10).

So I thought, lets quickly upgrade ... how hard can it be? A quick google and [nice easy to follow guide][guide_busterpi] and I was on my way to victory...

**Many** hours later ... ok it took a LOT longer than I was expecting but I finally upgraded to version 10 and tried again only to be sadly disappointed that it still refused to run. I shutdown my desktop and went to sleep because it was late into the night.

The next day after a good long night sleep I picked up where I left off. Could it be possible the script missed something? I recalled that Pete mentioned in the talk of the lengthy process to follow to install the SDK and get it running including a peice about setting the environment variables. The blog post was updated after that talk which had a nice script, so I scanned the post and found the details about updating your environment and voila I discovered the script does not include this.

So i quickly updated the ".bashrc" file to export the DOTNET_ROOT and tried to run my very basic project. A small tear and much happiness when I was finally greeted with: 

<hr class="wp-block-separator is-style-wide" />
<h1 class="has-text-align-center">Hello World!</h1>
<hr class="wp-block-separator is-style-wide" />

## Taming the Fire

[Webassembly][wikipedia_webassembly] has gained popularity which offers amazing benefits for creating Web Based applications that are almost on par with native desktop applications. And microsoft has not ignored this, working on its own variant, [Blazor][blazor].

Originally I was going to explore building my web app using Angular using typescript. Blazor is based on C# code working with Razor HTML formatting that behaves similar to Angular. When the code is published, some of the code is packaged into a library which is exposed through a javascript engine that acts like a bootloader of sorts.

This basically offers a similar experience to the C# MVC website but instead of running off the server, the application is running client side. Then using API calls to the backend server to facilitate your persistance and database needs. This could also be temporarily cached on the client, as in the case of a progressive web application.

## Summary

So this time around, this was more of a typical blog post and less of a technical posting. The project originally started as a goal of developing on a lightweight device (Windows Surface) to eventually host on a Raspberry Pi. It was somewhat of a parallel to what I was doing at work.

I am now reverting to my current favorite platform (C#) and to explore how it can compare other technologies.

Future projects I would like to explore is building up my own Pi Cluster to explore using things like Kubernetes and [OpenFaaS][openfaas]. I like the idea of writing very lean micro-services and being able to deploy a slim function mapped to an endpoint instead of a whole API. This offers a great flexibility where you don't need to re-deploy your entire API backend if changing/adding your API endpoints. FUN.

[Next Post][next_post]

[previous_post]: {% post_url 2020-02-29-my-web-app-journey-doing-an-oil-change %}
[next_post]: {% post_url 2020-08-29-my-web-app-journey-data-source-take-2 %}
[youtube_pete]: https://www.youtube.com/watch?v=l8CXgvKe314&amp;t=929s
[blog_pete]: https://www.petecodes.co.uk/explorations-in-dot-net-core-3-0-for-raspberry-pi/
[guide_busterpi]: https://pimylifeup.com/upgrade-raspbian-stretch-to-raspbian-buster/
[wikipedia_webassembly]: https://en.wikipedia.org/wiki/WebAssembly
[blazor]: https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor
[openfaas]: https://www.openfaas.com/blog/asp-net-core/