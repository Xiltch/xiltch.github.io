---
layout: post
title: My Web App Journey - Controllers
date: 2018-12-04 05:40:28.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
tags:
- MVC
- Typescript
author: Jonathan Tweedle
permalink: "/2018/12/04/my-web-app-journey-controllers/"
---
In the [previous chapter][previous_post] I refactored the design to break up the logic into multiple files while laying down the foundation for the controller pattern.

This chapter will focus on getting a controller that will implement basic CRUD operations that respond to different HTTP methods.

For now I will contain the data inside the controller design but later I want to try abstract to a data repository to manage the data IO and persistence.

## Show me the Money

Time to build out the controller that will handle the different CURD operations for a single transaction. Create a new typescript file in the controllers folder with just the bare bones for now.

#### controllers/transaction.ts
```typescript
import {Request, Response, Router} from 'express';

const router: Router = Router();

export const TransactionController: Router = router;
```

Next we need to map the controller to an endpoint to direct calls received to the controller. I updated the file in the main director that manages the routes to the now look like this.

#### routes.ts
```typescript
import {RootController} from './controllers/root';
import {TransactionController} from './controllers/transaction';

class Mapper {

  public addRoutes(app): void {
    app.use('/', RootController);
    app.use('/transaction', TransactionController);
  }

}

export const RouteMapper: Mapper = new Mapper();
```

The endpoint is now mapped but if you try to access it, you are only going to get an error because the controller does nothing for now. My first goal is to create a dummy array to store data and then introduce a method to handle a get that will return a given data element. The transaction controller now looks like this.

#### controllers/transaction.ts
```typescript
import {Request, Response, Router} from 'express';

const router: Router = Router();

var transactions = [
  { id: 1, type: "DEBIT", date: new Date('2018-11-28'), currency: 'USD', amount: -10.00, source: "DEBIT_CARD", description: "Soup" },
  { id: 2, type: "DEBIT", date: new Date('2018-11-28'), currency: 'USD', amount: -15.00, source: "DEBIT_CARD", description: "Dessert" },
  { id: 3, type: "DEBIT", date: new Date('2018-11-28'), currency: 'USD', amount: -20.00, source: "DEBIT_CARD", description: "Drinks" },
  { id: 4, type: "DEBIT", date: new Date('2018-11-28'), currency: 'USD', amount: -5.00, source: "DEBIT_CARD", description: "Tip" }
];

router.get('/:id', (req: Request, res: Response) => {
  let id = req.params.id;
  var transaction = transactions.find(x => x.id == id);

  if (transaction == null)
    res.status(404).send(); // Record not found
  else
    res.status(200).send(transaction);
});

export const TransactionController: Router = router;
```

One thing to note is that I did not specify in the controller the "/transaction/?" prefix because we handled that in the route mapping. So in the controller we treat the mapping as relative. The other part is extracting the parameters from the query string by labeling the value and then referencing it to filter my dummy data before returning it.

I can now compile my project and start it up and using Postman or even a web browser I submit a HTTP GET request to get one of the transactions.

![midas_201811282356][image_1]

## Giving Back

Now that we have implemented methods for getting a single transaction, the next step is creating a new transaction. This is achieved by using the HTTP POST method to send a transaction to the server to add to the list of transactions.

To post JSON formatted data to the server, requires using "body-parser" to enable parsing of JSON in the body of the request. So we need to update the app giving us the following.

#### app.ts
```typescript
import * as express from 'express';
import { RouteMapper } from './routes';
import * as bodyParser from 'body-parser';

class App {

    public app: express.Application;

    constructor() {
        this.app = express();
        this.config();
        RouteMapper.addRoutes(this.app);
    }

    private config(): void {
        this.app.use(bodyParser.json());
    }

}

export default new App().app;
```

With that in place we can now introduce a method to handle HTTP POST calls to the transaction controller. I also added a counter to handle unique id's to assign to new transactions that are added to the list. When the post method is called it creates a new transaction and takes the values from the body to create the values before pushing it on to the transaction list and then returns the created transaction with its id to confirm the transaction created successfully.

#### controllers/transaction.ts
```typescript
import {Request, Response, Router} from 'express';

const router: Router = Router();

var next_id = 5;

var transactions = [
  { id: 1, type: "DEBIT", date: new Date('2018-11-28'), currency: 'USD', amount: -10.00, source: "DEBIT_CARD", description: "Soup" },
  { id: 2, type: "DEBIT", date: new Date('2018-11-28'), currency: 'USD', amount: -15.00, source: "DEBIT_CARD", description: "Dessert" },
  { id: 3, type: "DEBIT", date: new Date('2018-11-28'), currency: 'USD', amount: -20.00, source: "DEBIT_CARD", description: "Drinks" },
  { id: 4, type: "DEBIT", date: new Date('2018-11-28'), currency: 'USD', amount: -5.00, source: "DEBIT_CARD", description: "Tip" }
];

router.get('/:id', (req: Request, res: Response) => {
  let id = req.params.id;
  var transaction = transactions.find(x => x.id == id);

  if (transaction == null)
    res.status(404).send(); // Record not found
  else
    res.status(200).send(transaction);
});

router.post('/', (req: Request, res: Response) => {
  
  var transaction = {
    id: next_id++,
    type: req.body.type, 
    date: new Date(req.body.date), 
    currency: req.body.currency, 
    amount: req.body.amount, 
    source: req.body.source, 
    description: req.body.description
  };
  
  transactions.push(transaction);

  res.status(200).send(transaction);

});

export const TransactionController: Router = router;
```

So now when we post some JSON data to the endpoint, we should get a 200 response with the same details with the new id included to confirm the record was created successfully. You can also try perform a GET call for the new id which should return your record.

![midas_201811292343][image_2]

## Have some Class

So I want to make updates to the transaction which includes classification to help organize them. My current design relies on a generic array that infers the properties of a transaction. The challenge this introduces is when it comes to adding new fields to accept new data.

To remedy this problem, I will start to model my data using classes to allow the declaration of future data elements not required initially but at a later stage. So we create a folder alongside the controllers called "models" and create inside a new class to define the Transaction.

One thing I had to make sure is to allow some parameters to be optional so that they don't need to specified when creating the array and it helps to keep the some of the code lean.

#### models/transaction.ts
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
```

With this change I can update the controller to use stronger typing and to implement a method to handle a HTTP PUT request. While I am at it, the final method supports deleting a transaction by using HTTP DELETE method call.

#### controllers/transaction.ts
```typescript
import {Request, Response, Router} from 'express';
import {Transaction} from '../models/transaction';

const router: Router = Router();

let next_id: number = 5;

let transactions: Array<Transaction> = [
  { id: 1, type: 'DEBIT', date: new Date('2018-11-28'), currency: 'USD', amount: -10.00, source: 'DEBIT_CARD', description: 'Soup' },
  { id: 2, type: 'DEBIT', date: new Date('2018-11-28'), currency: 'USD', amount: -15.00, source: 'DEBIT_CARD', description: 'Dessert' },
  { id: 3, type: 'DEBIT', date: new Date('2018-11-28'), currency: 'USD', amount: -20.00, source: 'DEBIT_CARD', description: 'Drinks' },
  { id: 4,type: 'DEBIT',date: new Date('2018-11-28'),currency: 'USD',amount: -5.00,source: 'DEBIT_CARD',description: 'Tip' }
];

router.get('/:id', (req: Request, res: Response) => {
  let id = req.params.id;
  let transaction = transactions.find(x => x.id == id);

  if (transaction == null)
    res.status(404).send();  // Record not found
  else
    res.status(200).send(transaction);
});

router.post('/', (req: Request, res: Response) => {

  let transaction: Transaction = {
    id: next_id++,
    type: req.body.type,
    date: new Date(req.body.date),
    currency: req.body.currency,
    amount: req.body.amount,
    source: req.body.source,
    description: req.body.description
  };

  transactions.push(transaction);

  res.status(200).send(transaction);

});

router.put('/:id', (req: Request, res: Response) => {

  let id: number = req.params.id;
  let transaction = transactions.find(x => x.id == id);

  if (transaction == null) 
    res.status(404).send();  // Record not found

  transaction.amount = req.body.amount || transaction.amount;
  transaction.currency = req.body.currency || transaction.currency;
  transaction.date = req.body.date || transaction.date;
  transaction.description = req.body.description || transaction.description;
  transaction.source = req.body.source || transaction.source;
  transaction.type = req.body.type || transaction.type;
  transaction.category = req.body.category;

  res.status(200).send(transaction);

});

router.delete('/:id', (req: Request, res: Response) => {

  let id: number = req.params.id;

  if (transactions.find(x => x.id == id) == undefined)
    res.status(404).send();  // Record not found

  transactions = transactions.filter(x => x.id != id);
  
  res.status(200).send('Transaction deleted');

});

export const TransactionController: Router = router;
```

I now have the basic methods in place to handle the CRUD functions for a transaction. The [next step][next_post] is work on integrating the methods with a MongoDB  to handle the persistence which I plan to tackle in the next chapter.

[previous_post]: {% post_url 2018-11-09-my-web-app-journey-wep-api-routing %}
[next_post]: {% post_url 2019-01-27-my-web-journey-data-repository %}
[image_1]: /assets/2018/12/midas_201811282356.jpg
[image_2]: /assets/2018/12/midas_201811292343.jpg