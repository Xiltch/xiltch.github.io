---
layout: post
title: My Web App Journey - Preparation
date: 2018-11-02 09:09:10.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
tags:
- Git
- My Web App Journey
- Node.js
author: Jonathan Tweedle
permalink: "/2018/11/02/my-web-app-journey-preparation/"
---
In the [previous chapter][previous_post] I introduced the project, in this chapter I will cover how I prepared my development environment for the project.

One of the goals is to perform the development on my Surface Pro which is running Windows 10 Pro with the intention of deploying to the Raspberry Pi (RPi) where it will run.

## Node.js Setup

That was the main reason to choose [Node.js][node_js] as the platform for running the application on top of. It can run on both Windows and Linux based platforms and its lightweight. I also need move the code back and forth between the two systems so I use [Git][download_git] which also helps in tracking the changes (good practice for any developer).

So the first step was downloading Node.js for windows and installing it to my Surface. At the time I was using the stable v8.12, but there should be no reason a later edition would not work. I am also a fan of [Microsoft's VS Code][download_vscode] which is a great lightweight code editor and works well for developing Node.js based applications.

For the RPi installation is rather simple.

```
sudo apt-get install nodejs
```

## Git Source Control Setup

Now with my goal to host the code on the RPi, I need a stable way to synchronize the code between my surface and the RPi. For this Git was the perfect choice as I don't need a cloud service to make it possible. I installed Git on my Surface and while optional I would also recommend the use of <a href="https://confluence.atlassian.com/get-started-with-sourcetree">Sourcetree </a>to manage local and remote repositories.

Next I setup Git on the RPi. You could try to install Gitlab but its not a requirement and you can achieve the same result with just the Git tools and SSH configuration.

```
sudo apt-get install git
```

Once installed you can create your repos on your RPi. I chose to create a directory on a memory stick to house all my repos (might as well use it for other projects as well) which I later soft linked to the root direct to shorten the git URLs when doing a checkout from remote machines.

Now to create a bare git repository under my "repos" directory.

```
git init --bare projectName
```

If you look inside the directory you will see a bunch of directories, you cant work directly inside this directory so you need to navigate to a directory where you intend to run your copy from and checkout the git project.

```
git clone ssh://piAccount@192.168.0.30:/repos/projectName
```

I do this on both the RPi and on my Surface which allows me synchronize changes between both systems. As mentioned previously, I make use of Sourcetree on my Windows system to manage git changes.

**Example:**
```
git pull origin master
git add .
git commit -m 'checking in recent changes'
git push origin master
```

## Mongo DB Setup

While the later version of Mongo is available, sadly I have to install version 2.4 because at the time of this blog there is no 32 bit after version 2.4 available on the RPi.

So you could [download][download_mongodb] the 64 bit version for Windows but you could run into problems where you got something working in the dev environment but once on the RPi it fails to work.

Installation once again on the RPi is straight forward.

```
sudo apt-get install mongodb
```

You can go a step further and change the config to store your database on a mounted drive or flash stick to preserve the life of the sd card the OS is running on.

In the [next chapter][next_post], we start with the WEB api.

[previous_post]: {% post_url 2018-10-31-my-web-app-journey-introduction %}
[next_post]: {% post_url 2018-11-08-my-web-app-journey-beginnings-of-the-web-api %}
[node_js]: https://nodejs.org/
[download_git]: https://git-scm.com/download/win
[download_vscode]: https://code.visualstudio.com/
[download_mongodb]: https://www.mongodb.com/download-center/community