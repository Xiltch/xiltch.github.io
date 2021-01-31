---
layout: post
title: My Web App Journey - Beginnings of the Web API
date: 2018-11-08 09:40:38.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
tags:
- My Web App Journey
- Node.js
- Typescript
author: Jonathan Tweedle
permalink: "/2018/11/08/my-web-app-journey-beginnings-of-the-web-api/"
---
In the [previous chapter][previous_post] I got my development environment ready to start the development process. In this chapter I start working on the Web API.

The purpose of the Web API is provide a standardize method for interacting with the data used by Web App. This is not just limited to just Web App's but can also be used in desktop applications or services that have a need to interact with data in different ways.

While you could directly connect to say a SQL database, if your design runs into scaling issues and you decide to change over to a NoSQL based data store it would require rewriting a lot of your code to wire in new changes. If you have multiple applications using the same data source this means having to update each of them. By adding the API layer you can safely switch out your underlying data store without impacting the rest of your stack.

## Starting a new project

So to recap in the previous chapter we first had to create a new git repos, I did this by logging into my Raspberry Pi and created a directory for my project and initialized it as a bare git repository.

```
cd /repos
mkdir Midas
cd /repos/Midas
git init --bare
```

On my Windows system I open up the Git Bash prompt and navigate to my local workspace folder (a general folder to house multiple projects) and clone my new empty project.

```
cd /c/workspace/
git clone ssh://git@192.168.0.30/repos/Midas
```

Now it is pretty lonely in there so before we go about adding stuff the first thing i want is to add a ".gitignore" file to the project. This file tells git to ignore files that match the pattern from being committed into the repository, this helps to keep the repository lean and free of junk such as compiled output and downloaded libraries.

If you are using VS Code, you can shortcut straight from the git bash console you already have open.

```
cd Midas
code .gitignore
```

Now using your text editor of choice add the following and save the file.

```
node_modules/
package-lock.json
dist/
```

You can now go ahead and commit the changes to your git repository.

```
git add .
git commit -m 'created ignore file'
git push origin master
```

Going forward I will assume that you are committing your code into your repository after changes.

## Initializing the Node.js Project

Lets now get the project started and initialize our Node.js project in our somewhat empty project directory.

```
npm init
```

Hopefully you have node installed and it will kick of the wizard. You will need to give your project name (I called mine 'midas'). You will also need to give it a version number, a description, etc ...

I stuck to the defaults except for the description but the defaults are fine for now. Once done and you accept, it will generate the package.json file. We are going to update this file over time to either directly or through npm commands as the project evolves.

Time to add some dependencies to our project, the following commands will download the require libraries and update the package.json file.

```
npm install express
npm install nodemon --save-dev
npm install typescript --save-dev
npm install @types/node --save-dev
npm install @types/express --save-dev
npm install @types/body-parser --save-dev
```

So because I have now declared my intent to use typescript ... i need to configure the project to use it. The quickest way I currently know is to create a 'tsconfig.json' in the root directory of the project, with the following contents:

```json
{
    "compileOnSave": true,
    "compilerOptions": {
      "target": "es6",     //default is es5
      "module": "commonjs",//CommonJs style module in output
      "outDir": "dist"  ,   //change the output directory
      "resolveJsonModule": true //to import out json database
    },
    "include": [
      "src/**/*.ts"       //which kind of files to compile
    ],
    "exclude": [
      "node_modules"     //which files or directories to ignore
    ]
 }
```

This instructs the typescript compiler (tsc) how to handle compiling typescript into javascript. If you have not yet installed typescript just run the following.

```
npm install -g typescript
```

You might have also noticed I told the compiler to include all typescript files (*.ts) located under the "src" directory so lets go ahead create the directory in the root directory. So we should now have the following structure.

```
/
|-- .git/
|-- node_modules/
|-- src/
|-- .gitignore
|-- package.json
|-- package-lock.json
|-- tsconfig.json
```

The structure is going to evolve over time as the application evolves and while you could treat these as individual projects, my goal is to house them together in one repository.

## Hello World

Now on to the meat of the API and what better way than to start with a simple "Hello World" to make sure the environment is working correctly

Lets start by creating "server.ts" under the "src/" directory, and adding some simple code.

```
console.log("Hello World");
```

At this point you should be able to compile your typescript and test it works by executing the following in the root of the project directory.

```
tsc
node dist/server.js
```

While this works, lets be lazy and setup some scripts to automate some things. We do this in the "package.json" file located in the root of the project.

Look for the line ' "main": "index.js" and change it to the following.

```json
"main": "./dist/server.js"
```

You should should also find a scripts section that we update with some scripts which will trigger the typescript compiler to take our ".ts" files and convert them into the '.js" files in the "dist/" directory before starting our simple application.

```json
"scripts": {
  "build" : "tsc",
  "prestart": "npm run build",
  "start": "node ./dist/server.js"
}
```

Now you can do it all with one simple command.

```
npm start
```

If all is working well, you should see the compiler run followed by the expected output of "Hello World". Now instead of sending it to the console, we want to send it to a browser in response to web request So lets update the "server.ts" file to the following.

```typescript
import * as express from 'express';

const PORT = 3000;
const HOST = 'localhost';
var app: express.Application;

app = express();

app.get('/', function(req, res) {
  console.log('client connected');
  res.send('Hello World');
});

app.listen(PORT, HOST, () => {
  console.log('listening on port ' + PORT);
});
```

Once it has updated run "npm start" and when you see the server is listening, navigate to [http://127.0.0.1:3000/][npm_start] and you should get a response of "Hello World" and in your console you should see a message that a client connected.

In the [next chapter][next_post], I will refactor the solution to abstract the design with routing and controllers to manage requests.

[previous_post]: {% post_url 2018-11-02-my-web-app-journey-preparation %}
[next_post]: {% post_url 2018-11-09-my-web-app-journey-wep-api-routing %}
[npm_start]: http://127.0.0.1:3000/