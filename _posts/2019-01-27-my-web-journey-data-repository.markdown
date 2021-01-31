---
layout: post
title: My Web App Journey - Going Solid
date: 2019-01-27 09:35:50.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
tags:
- Data Repository
- Dependency Inversion
- Node.js
- Typescript
author: Jonathan Tweedle
permalink: "/2019/01/27/my-web-journey-data-repository/"
---
In the [previous chapter][previous_post] I made some progress in the design of the controller to handle the basic CRUD operations for a transaction.

The project will grow to include more controllers but right now the transaction controller is lacking data persistence. Now I could design the controller to just connect directly to a database of my choice (such as MongoDB) however I would prefer using the repository pattern to abstract the data implementation.

I also plan on shifting my approach in the direction of Domain Driven Design (DDD) with some SOLID principles. So this chapter will be a detour before adding MongoDB support.

## Becoming Independent

I have recently been looking at content on dependency injection and dependency inversion. The goal is to further decouple the design and abstract the design. So I want to have the concept of a repository and my application to make use of this generic notion but never actually instantiating an instance but instead have it "injected" in wherever it is needed. 

I plan to rely mainly on Inversion, to avoid concretions that tightly couple the design of one aspect to another. Of course there is still a need at some point where the concretions need to be defined and mapped together to make the application work. This will be made possible using the [Inversify][github_inversify] module. (It also means this post is going to be a long one).

_I decided to rely mainly on Inversion, to avoid concretions that tightly couple the design of one aspect to another. Of course there is still a need at some point where the concretions need to be defined and mapped together to make the application work. This will be made possible using the Inversify module._

Install it by running the command in the root of the project
```
npm install inversify reflect-metadata --save
```

Inversify relies on some black magic to make the injection possible which is not enabled by default. You need to inform the typescript compiler to enable the use of experimental decorators and to include the types from the reflect-metadata module. This gives us the following:

#### /tsconfig.json
```json
{
    "compileOnSave": true,
    "compilerOptions": {
      "experimentalDecorators": true,
      "emitDecoratorMetadata": true,
      "moduleResolution": "node",
      "types": ["reflect-metadata"],
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

With that out of the way I can focus on making my dream come true. As mentioned earlier the concretions get managed from a single point in the project.

## Tidying up

There are a few changes I need to make to the project t along with the use of templates to help generalize the design. Rather than relying on concretions, I first need to define an interface to describe how to interact with a repository for the transactions. I could create a new file for the interface but chose to append it to the existing model for the transaction as it is technically tightly coupled to the design of a transaction. I included some [TypeDoc][typedoc] comments to improve the documentation in the intellisense to know the expected behavior any repository implementing the interface will honor.

_To make it possible, I need to make use of templates to help generalize the design. So rather than relying on concretions, I first need to define an interface to describe how to interact with a repository for the transactions. I could create a new file for the interface but chose to append it to the existing model for the transaction as it is technically tightly coupled to the design of a transaction. I included some TypeDoc comments to track this._

#### /src/models/transaction.ts
```typescript
export class Transaction {
  id: number;
  amount: number;
  currency: string;
  date: Date;
  description: string;
  source: string;
  type: string;
  category?: string;
}

export interface TransactionRepository {
  /** finds the first matching Transaction in the repository
   * @returns transaction. null if not found
   */
  find(filter: (Transaction) => boolean): Transaction;
  /** add a new transaction to the repository
   * @returns transaction. null if it failed to add
   */
  add(record: Transaction): Transaction;
  /** updates a transaction in the repository
   * @returns transaction. null if unsuccessful */
  update(record: Transaction): Transaction;
  /** removes a record from the repository
   * @returns transaction. null if unsuccessful */
  remove(filter: (Transaction) => boolean): Transaction;
  /** finds all the matching transactions in the repository
   * @returns array of transaction. null if nothing found
   */
  findAll(filter: (Transaction) => boolean): Array<Transaction>;
}
```

From there I need to define some symbols that will be used for handling the dependency injection. These will later provide the means to manage the binding to the instances that will be injected when the application is running.

#### /src/types.ts
```typescript
export const REPOSITORY_TYPES = {
  Transaction: Symbol.for('Transactions')
};

export const CONTROLLERS = {
  Transaction: Symbol.for('Transaction')
};

export const SINGLETONS = {
  Routing: Symbol.for('Routing')
};
```

I now need to update the controller to move the data array out and replace  it with an injected repository. I also need to encapsulate it into a class which will allow the controller be instantiated at run time as it gets injected.

#### /src/controllers/transaction.ts
```typescript
import {Request, Response, Router} from 'express';
import {Transaction, TransactionRepository} from '../models/transaction';
import { REPOSITORY_TYPES } from '../types';
import { inject, injectable } from "inversify";

@injectable()
export class TransactionController {

  private repository: TransactionRepository;

  private router: Router = Router();

  public constructor(@inject(REPOSITORY_TYPES.Transaction) repository: TransactionRepository) {

    this.repository = repository;

    this.router.get('/:id', (req: Request, res: Response) => {

      let id = req.params.id;
      let transaction = this.repository.find(x => x.id == id);
    
      if (transaction == null)
        res.status(404).send();  // Record not found
      else
        res.status(200).send(transaction);
    });
    
    this.router.post('/', (req: Request, res: Response) => {
    
      let transaction: Transaction = {
        id: 0,
        type: req.body.type,
        date: new Date(req.body.date),
        currency: req.body.currency,
        amount: req.body.amount,
        source: req.body.source,
        description: req.body.description
      };
    
      this.repository.add(transaction);
    
      res.status(200).send(transaction);
    
    });
    
    this.router.put('/:id', (req: Request, res: Response) => {
    
      let transaction = this.repository.update({
        id:  req.params.id,
        amount:  req.body.amount,
        currency:  req.body.currency,
        date:  req.body.date,
        description:  req.body.description,
        source:  req.body.source,
        type:  req.body.type,
        category:  req.body.category
      });
    
      if (transaction == null) 
        res.status(404).send();  // Record not found

      res.status(200).send(transaction);
    
    });
    
    this.router.delete('/:id', (req: Request, res: Response) => {
    
      let id: number = req.params.id;
    
      if (this.repository.find(x => x.id == id) == undefined)
        res.status(404).send();  // Record not found
    
      this.repository.remove(x => x.id != id);
      
      res.status(200).send('Transaction deleted');
    
    });
  }
  
  public getRouter() : Router {
    return this.router;
  }

}
```

## Getting Real

So the next step is to implement the TransactionRepository interface. Using the array from the controller I can create a basic repository. This will allow it to be injected into the design. This design will also allow me to swap out one repository for another without having to change any other code (unless I change my interface design). 

#### /src/repositories/transactionArray.ts
```typescript
import {injectable} from 'inversify';

import {Transaction, TransactionRepository} from '../models/transaction';

@injectable()
export class TransactionArrayRepository implements TransactionRepository {
  private next_id: number = 5;

  private transactions: Array<Transaction> = [
    { id: 1, type: 'DEBIT', date: new Date('2018-12-28'), currency: 'USD', amount: -10.00, source: 'DEBIT_CARD', description: 'Soup' },
    { id: 2, type: 'DEBIT', date: new Date('2018-12-28'), currency: 'USD', amount: -15.00, source: 'DEBIT_CARD', description: 'Dessert' },
    { id: 3, type: 'DEBIT', date: new Date('2018-12-28'), currency: 'USD', amount: -20.00, source: 'DEBIT_CARD', description: 'Drinks' },
    { id: 4,type: 'DEBIT',date: new Date('2018-12-28'),currency: 'USD',amount: -5.00,source: 'DEBIT_CARD',description: 'Tip' }
  ];

  public find(evaluator: (Transaction) => boolean): Transaction {
    return this.transactions.find(evaluator);
  }

  public findAll(evaluator: (Transaction) => boolean): Array<Transaction> {
    if (evaluator == null) return Object.assign([], this.transactions);

    return this.transactions.filter(evaluator);
  }

  public add(record: Transaction): Transaction {
    let entry = Object.assign({}, record);
    entry.id = ++this.next_id;
    this.transactions.push(entry);
    return entry;
  }

  public update(record: Transaction): Transaction {
    let entry = this.find(x => x.id == record.id);
    if (entry == null) return null;
    let updatedEntry = Object.assign(entry, record);
    return updatedEntry;
  }

  public remove(evaluator: (Transaction) => boolean): Transaction {
    this.transactions = this.transactions.filter(evaluator);
    return null;
  }
}
```

## Time to Re-Route things

With the changes to the TransactionController I need to tweak the route mapping. I also need to tweak the design to make it inject-able. It's not fully in line with SOLID but its a step in the right direction.

#### /src/routes.ts
```typescript
import {RootController} from './controllers/root';
import {TransactionController} from './controllers/transaction';
import { injectable, inject } from 'inversify';
import { CONTROLLERS } from './types';

@injectable() 
export class Mapper {

  private _TransactionController: TransactionController;

  public constructor(@inject(CONTROLLERS.Transaction) transactionController: TransactionController) {
    this._TransactionController = transactionController;
  }

  public addRoutes(app): void {
    app.use('/', RootController);
    app.use('/transaction', this._TransactionController.getRouter());
  }

}
```

## Binding things together

Dependency injection relies on mapping the types to the class implementation. This is what allows the ability of implementing say a new repository without having to update every its needed, instead I just update the mapping in a single place.

#### /src/inversify.config.ts
```typescript
import "reflect-metadata";
import {Container} from 'inversify';
import {TransactionController} from './controllers/transaction';
import {TransactionRepository} from './models/transaction';
import {TransactionArrayRepository} from './repositories/transactionArray';
import {Mapper} from './routes';
import { CONTROLLERS, SINGLETONS, REPOSITORY_TYPES } from "./types";

const container = new Container();
container.bind<TransactionController>(CONTROLLERS.Transaction).to(TransactionController);
container.bind<Mapper>(SINGLETONS.Routing).to(Mapper);
container.bind<TransactionRepository>(REPOSITORY_TYPES.Transaction).to(TransactionArrayRepository);

export {container};
```

## The final piece


Time to wrap it up all together in the main entry point where the single instance of inversify config gets created that is used and to ensure the mappings are performed.

#### /src/app.ts
```typescript
import * as express from 'express';
import { Mapper } from './routes';
import * as bodyParser from 'body-parser';
import { SINGLETONS } from './types';
import { container } from './inversify.config';

class App {

    public app: express.Application;
    private RouteMapper: Mapper =  container.get<Mapper>(SINGLETONS.Routing);

    constructor() {
        this.app = express();
        this.config();
        this.RouteMapper.addRoutes(this.app);
    }

    private config(): void {
        this.app.use(bodyParser.json());
    }

}

export default new App().app;
```

With those changes, not much appears to have changed on the surface. In the [next chapter][next_post] I will write a new repository class that works with MongoDB and change out the old one.

## Contributing Research Material

<span style="font-size:12px;"><a rel="noreferrer noopener" href="https://hackernoon.com/generic-repository-with-typescript-and-node-js-731c10a1b98e" target="_blank">Wendel,  Erik.  Patterns — Generic Repository with Typescript and Node.js. Web Article. 20 March 2018.</a></span>

<span style="font-size:12px;"><a rel="noreferrer noopener" href="https://dev.to/remojansen/implementing-the-onion-architecture-in-nodejs-with-typescript-and-inversifyjs-10ad" target="_blank">Jansen, Remo H. Implementing SOLID and the onion architecture in Node.js with TypeScript and InversifyJS. Web Article. 13 April 2018.</a></span>

<span style="font-size:12px;"><a href="https://itnext.io/typescript-dependency-injection-setting-up-inversifyjs-ioc-for-a-ts-project-f25d48799d70" target="_blank" rel="noreferrer noopener">Abrickis, Andres. Typescript dependency injection: setting up InversifyJS IoC for a TS project. Web Articel. 9 July 2018.</a></span>

<span style="font-size:12px;"><a rel="noreferrer noopener" href="https://www.youtube.com/watch?time_continue=669&amp;v=_lwCVE_XgqI" target="_blank">Taylor, Jason. Clean Architecture with ASP.NET Core 2.1. YouTube. 18 October 2018.</a></span>


[previous_post]: {% post_url 2018-12-04-my-web-app-journey-controllers %}
[next_post]: {% post_url 2020-02-29-my-web-app-journey-data-store %}
[github_inversify]: https://github.com/inversify/InversifyJS
[typedoc]: http://typedoc.org/guides/doccomments/