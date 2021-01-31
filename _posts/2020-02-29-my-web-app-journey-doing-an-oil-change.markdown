---
layout: post
title: My Web App Journey - Doing an oil change
date: 2020-02-29 19:49:09.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
tags:
- Angular
- My Web App Journey
- Node.js
- Upgrade
author: Jonathan Tweedle
permalink: "/2020/02/29/my-web-app-journey-doing-an-oil-change/"
---
As mentioned in my [last post][previous_post], I took a bit of a break to focus on some studies and then relocated half way around the world. In that time some of the core technologies I use for the project have evolved as is normally the case. Even the way in which posts in WordPress are authored has changed a little.

I decided this would be would be great motivation for me to upgraded my development environment and work through all the issues it brings with it that impact the project.

### Rocky start


So i normally would prefer to start with a fully working state in my code but I suspect i will have many new issues to resolve after I upgrade and it might result in further changes to the underlying design so while it is tempting to first fix the code I will instead download and install the latest environments.

I started by installing NodeJs 64 bit and opted to also install Chocolatey which is basically a software package manager platform that can help with installing of additional software. If you want to be a purist then you can avoid that and just install the core pieces manually.

 After a reboot, I opened a command prompt and navigated to the root folder of my project to upgrade some of the packages and dependencies (like Typescript).

Upgrading typescript to the latest is straight forward through the use of the following npm command.

```
npm install -g typescript@latest
```

This updates the global package and not the project so I have to update the package as well. I don't know if i could have skipped updating the global package but it did not hurt.  In fact it turns out you can quickly update all the packages in your solution.

```
npm update
npm install
```

That sorts out the project as I have not yet started using Angular so luckily I have one less thing to update. That leaves updating MongoDb to the current version. This is made a little easier with Chocolatey. Open a command prompt with Administrative privileges.

```
choco install mongodb
```

This automates installing the latest version and updating the windows service to make sure it is running. If you would prefer to manually control the database server then you can go and change the service from automatic to manual. You will also need to update your local path settings if you have an older version installed to make sure the commands resolve to the latest version.

I have now run into the first issue. Upgrading from MongoDB 3.4 straight to 4.2 left  me with a database which is not supported with the new version. If i want to keep my existing data I would need to follow the following upgrade path, 3.4 => 3.6 => 4.0 => 4.2.

That sounds like a little too much hard work and as I don't yet have any data I am willing to wipe and reset. I simply deleted the old db directory and created a new one, fired up the mongo server in standalone mode and it magically created a fresh database.

While i might not yet have an Angular web app I might as well update the CLI at the same time. This is easily done through the use of NPM.

```
npm install -g @angular/cli
```

I am now ready to fix the problems the past me left for the [future me to solve][next_post]. 

### Contributing Research Material

<span style="font-size:12px;"><a href="https://stackoverflow.com/questions/39677437/how-to-update-typescript-to-latest-version-with-npm">Eaviden. How to update typescript to latest version with npm. Stack overflow. 24 September 2016.</a></span>

<span style="font-size:12px;"><a href="https://chocolatey.org/packages/mongodb">mkevenaar, Chocolatey packages page for MongoDB. Website. 29 February 2020</a></span>

[previous_post]: {% post_url 2020-02-29-my-web-app-journey-data-store %}
[next_post]: {% post_url 2020-08-23-my-web-app-journey-taming-the-fire %}