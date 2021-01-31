---
layout: post
title: My Web App Journey - Wep API Routing
date: 2018-11-09 10:17:44.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
tags:
- My Web App Journey
- Typescript
author: Jonathan Tweedle
permalink: "/2018/11/09/my-web-app-journey-wep-api-routing/"
---
In the [previous chapter][previous_post] I initialized the node project and configured it to use Typescript. I finished it off by creating a simple express application to respond to web requests.

In this project I will refactor the design and move things around a little and add some routing to handle more complicated design. Another goal is to migrate the design to something that is hopefully more scalable in its design for future iterations.

Now before I start, while you can use a browser to connect to the API it does not offer all the tools to design a Web API. I recommend the use of a software package like [PostMan][download_postman] which really helps when developing your different API calls.

## Be more abstract

Taking some design from the angular method, they split the server implementation from the definition into two separate files. As we already have the integration file, we need to move the design pieces over into the "app.ts" file and leave the "server.ts" more lean so that I have this.

#### server.ts

```typescript
import app from './app';

const PORT = 3000;
const HOST = 'localhost';

app.listen(PORT, HOST, () => {
  console.log('listening on port ' + PORT);
});
```

For this to work I need to make sure that in the app.ts file I create an instance of the express application and set it up so that in our server file we focus on telling the application to listen for new connections. I settled on the following to a more [OOP][wikipedia_oop] style approach and wrapping the express application in my own Class definition.

#### app.ts

```typescript
import * as express from 'express';

class App {

  public app: express.Application;

  constructor() {
     this.app = express();
     this.config();
  }

  private config(): void {

    this.app.get('/', function(req, res) {
        console.log('client connected');
        res.send('Hello World');
    });
      
  }

}

export default new App().app;
```

At this point I am getting tired of running ```npm start``` every time I make a change to my code. Thankfully there is some nice tools to help with this problem. I am going to use ```nodemon``` and ```ts-node``` to monitor for changes in the src while I am developing and if there are changes, it will restart using the new ts files.

I already had nodemon installed in a previous step so all I needed was to add ts-node to my list of dev dependencies. The easiest way is with the following command.

```
npm install ts-node --save-dev
```

Now I have to add a new script that I can use to call nodemon so I open the package.json file in the root directory and update the scripts to include my new script.

```json
"dev": "nodemon --config nodemon-dev.json"
```

As you might notice this relies on a configuration file called ```nodemon-dev.json``` which should be located in the root folder next to our ```package.json``` file with the following details.

#### nodemon-dev.json

```json
{
    "watch": ["src"],
    "ext": "ts",
    "ignore": ["src/**/*.spec.ts"],
    "exec": "ts-node ./src/server.ts"
}
```

And with that I can run ```npm run dev``` and it will restart the server as I update the code.

## Becoming more controlling

So our API is coming along nicely however we are still a little heavy in a single point of the project, the app definition. As we expand the API to include new endpoints with different GET/POST requests, it will become very bloated.

I like the controller pattern from MVC C# solutions but also want something modular should I want to use the controllers in other projects.

So I add a new directory under the src folder called ```controllers``` that will host each of the controllers I want to build out.

I can now create a new controller to handle the root path of the web API and call it ```root.ts```. Looking at the src directory I now have the following layout.

![midas_201811082319][image_1]

To make the root controller work, it needs to create a router for the request similar to what I have currently in the ```app.ts``` file. I will later take use the resulting router to map it to the main application.

#### root.ts

```typescript
import {Request, Response, Router} from 'express';

const router: Router = Router();

router.get('/', (req: Request, res: Response) => {
    console.log("Got a new connection");
    res.status(200).send('Welcome to the Midas API server');
});

export const RootController: Router = router;
```

Next I need to wire the routes for my endpoints between the controllers and the main application. Once again, I want to keep the bloat out of the main app file so I created a separate file to handle the route mapping.

#### routes.ts

```typescript
import {RootController} from './controllers/root';

class Mapper {

  public addRoutes(app): void {
    app.use('/', RootController);
  }

}

export const RouteMapper: Mapper = new Mapper();
```

All that is needed now is to update the app definition to import my route mapper while removing the now irrelevant get call. So now this is what the app definition looks like.

#### app.ts

```typescript
import * as express from 'express';
import { RouteMapper } from './routes';

class App {

    public app: express.Application;

    constructor() {
        this.app = express();
        this.config();
        RouteMapper.addRoutes(this.app);
    }

    private config(): void {

    }

}

export default new App().app;
```

I am now ready to scale out the different endpoints of my API by creating a controller for each endpoint and then mapping it to a given route. The one benefit of this design is when designing the controllers the paths are relative so if you want to change the route, you can do it by updating a single line.

In the [next chapter][next_post] I will create dummy controller with various GET and POST handlers that will get mapped to a route. It will also return some dummy data in a json format.

[previous_post]: {% post_url 2018-11-08-my-web-app-journey-beginnings-of-the-web-api %}
[next_post]: {% post_url 2018-12-04-my-web-app-journey-controllers %}
[download_postman]: https://www.getpostman.com/
[wikipedia_oop]: https://en.wikipedia.org/wiki/Object-oriented_programming
[image_1]: /assets/2018/11/midas_201811082319.png