---
layout: post
title: My Web App Journey - Data Source (Take 2)
date: 2020-08-29 18:01:01.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
tags:
- Data Repository
- MongoDB
- My Web App Journey
- Typescript
author: Jonathan Tweedle
permalink: "/2020/08/29/my-web-app-journey-data-source-take-2/"
---
I did the equivalent of an oil change in a [previous post][previous_post] to upgrade my project to use the latest versions of the different packages to minimize the technical debt having halted the development for about a year. 

I was in the middle of [changing the data source][post_data_storev1] which  has left the project broken because the both the original repository that leveraged an in-memory array and the new repository that works with the MongoDB server no longer match the changes I made to the interface. 

Typically you should create a new version of your interface to avoid breaking changes, because the result is I now need to update not just both repositories but also the controller to leverage the new design. This is the cost of  trying to shift all the control from the repository directly into the controller. 

While reviewing the interface design which is now less generic making the coupling between the controller and the data nice and loose, it suffers from being a single use. 

### Building the chain

So by "single use", I am referring to the ability to use a function from the repository to return a list of items as limited to a single step. A better way to think of this would be wanting to get from the database a list of items created on a specific date which are categorized as food expenses. 

Currently my interface is such that you would need to choose the primary filter to the items but then you would need to perform additional filtering inside the controller. This feels like additional work to me. What would be nice is to create something that is more like a LINQ style function found in C#. I don't know if its possible and I might end up simulating the behavior described above but in the repository itself.

I started by trying to explore the mongodb client to understand its capabilities but found I lacked the types. So first I need to install those.

```
npm install @types/mongodb
```

Going back into vs code, I leveraged the linting to explore the functions used on mongo client as to the parameters and return type. Thinking about how LINQ is achieved inside the C# language might help, instead of the "Find" methods return the list of objects they would need to return some sort of Type that feeds into other "Find" methods with a final method to process and "Collect" the items from the database.

The challenge is to design it in such a way that is repository agnostic to avoid bias towards a particular database. The solution I plan to settle on will be a new interface that specifies a bunch of different filter methods. Each filter method returns the very same interface to allow chaining methods. A final "Collect" method being the bookend which then returns a collection of items.

**/src/models/transaction.ts**
```typescript
export class Transaction {
  id: string;
  amount: number;
  currency: string;
  date: Date;
  description: string;
  source: string;
  type: string;
  category?: string;
}

export interface TransactionRepositoryFilter {

  /** find transactions in a category. supports regular expression matching.
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByCategory(category: string): TransactionRepositoryFilter;

  /** find transactions based on the description. supports regular expression matching.
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByDescription(description: string): TransactionRepositoryFilter;

  /** find transactions based on the source. supports regular expression matching.
   * @returns instance of a TransactionRepositoryFilter
  */
  filterByBySource(source: string): TransactionRepositoryFilter;

  /** find transactions based on the type. supports regular expression matching.
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByType(type: string): TransactionRepositoryFilter;

  /** find transactions between two given amounts (inclusive).
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByAmountBetween(lower: number, upper: number): TransactionRepositoryFilter;

  /** find transactions above a given amount (exclusive).
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByAmountAbove(amount: number): TransactionRepositoryFilter;

  /** find transactions below a given amount (exclusive).
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByAmountBelow(amount: number): TransactionRepositoryFilter;

  /** find transactions at a given amount (inclusive).
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByAmount(amount: number): TransactionRepositoryFilter;

  /** find transactions between two given dates (inclusive).
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByDateBetween(lower: Date, upper: Date): TransactionRepositoryFilter;

  /** find transactions after a given date (exclusive).
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByDateAfter(date: Date): TransactionRepositoryFilter;

  /** find transactions before a given date (exclusive).
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByDateBefore(date: Date): TransactionRepositoryFilter;

  /** find transactions on a given amounts.
  * @returns instance of a TransactionRepositoryFilter
  */
  filterByDate(date: Date): TransactionRepositoryFilter;

  /** collect all Transactions based on the previous filters
   * @returns array of transactions. null if no records found
   */
  collect(): Array<Transaction>;

}

export interface TransactionRepository extends TransactionRepositoryFilter {

  /** add a new transaction to the repository.
  * @returns transaction. null if it failed to add
  */
  add(record: Transaction): Transaction;

  /** updates a transaction in the repository.
  * @returns transaction. null if unsuccessful 
  */
  update(record: Transaction): Transaction;

  /** get all the transactions in the repository.
  * @returns array of transaction. null if no records found
  */
  all(): Array<Transaction>;

  /** finds the first matching Transaction in the repository based on its ID. supports regular expression matching.
  * @returns transaction. null if not found
  */
  getById(id: string): Transaction;

  /** removes a record from the repository.
  * @returns transaction. null if unsuccessful */
  removeById(id: string): Transaction;

}
```

I think that should do it, how exactly I manage to make it actually happen will be down to the implementation of each repository. 

## Housekeeping

With the new tweaks to the interface, need to update the controller to take advantage. This means removing the expression logic from the controller to focus more on translating the incoming request, calling the relevant repository method and generating a response. 

**/src/controllers/transaction.ts**
```typescript
import { Request, Response, Router } from 'express';
import { Transaction, TransactionRepository } from '../models/transaction';
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
      let transaction = this.repository.getById(id);

      if (transaction == null)
        res.status(404).send();  // Record not found
      else
        res.status(200).send(transaction);
    });

    this.router.post('/', (req: Request, res: Response) => {

      let transaction: Transaction = {
        id: "0",
        type: req.body.type,
        date: new Date(req.body.date),
        currency: req.body.currency,
        amount: req.body.amount,
        source: req.body.source,
        description: req.body.description
      };

      transaction = this.repository.add(transaction);

      if (transaction == null)
        res.status(404).send();  // Record not added
      else
        res.status(200).send(transaction);

    });

    this.router.put('/:id', (req: Request, res: Response) => {

      let transaction = this.repository.update({
        id: req.params.id,
        amount: req.body.amount,
        currency: req.body.currency,
        date: req.body.date,
        description: req.body.description,
        source: req.body.source,
        type: req.body.type,
        category: req.body.category
      });

      if (transaction == null)
        res.status(404).send();  // Record not found
      else
        res.status(200).send(transaction);

    });

    this.router.delete('/:id', (req: Request, res: Response) => {

      let id = req.params.id;
      let transaction = this.repository.removeById(id);

      if ( transaction == null)
        res.status(404).send();  // Record not found
      else
        res.status(200).send('Transaction deleted');

    });
  }

  public getRouter(): Router {
    return this.router;
  }

}
```

I updated the repository based on the array and fired up the project to make sure it was still functional following all the changes I had made. I found a bug with the update and quickly touched that up.

**/src/repositories/transactionArray.ts**
```typescript
import { injectable } from 'inversify';

import { Transaction, TransactionRepository } from '../models/transaction';

@injectable()
export class TransactionArrayRepository implements TransactionRepository {

  private next_id: number = 5;

  private transactions: Array<Transaction> = [
    { id: "1", type: 'DEBIT', date: new Date('2018-12-28'), currency: 'USD', amount: -10.00, source: 'DEBIT_CARD', description: 'Soup' },
    { id: "2", type: 'DEBIT', date: new Date('2018-12-28'), currency: 'USD', amount: -15.00, source: 'DEBIT_CARD', description: 'Dessert' },
    { id: "3", type: 'DEBIT', date: new Date('2018-12-28'), currency: 'USD', amount: -20.00, source: 'DEBIT_CARD', description: 'Drinks' },
    { id: "4", type: 'DEBIT', date: new Date('2018-12-28'), currency: 'USD', amount: -5.00, source: 'DEBIT_CARD', description: 'Tip' }
  ];

  public getById(id: string): Transaction {
    return this.transactions.find(x => x.id == id);
  }

  public all(): Array<Transaction> {
    return Object.assign([], this.transactions);
  }

  public add(record: Transaction): Transaction {
    let entry = Object.assign({}, record);
    entry.id = (++this.next_id).toString();
    this.transactions.push(entry);
    return entry;
  }

  public update(record: Transaction): Transaction {
    let entry = this.getById(record.id);
    if (entry == null) return null;
    Object.keys(record).forEach(prop => {
      if (record[prop]) {
        entry[prop] = record[prop];
      }
    });
    return entry;
  }

  public removeById(id: string): Transaction {
    let entry = this.getById(id);
    if (entry == null) return null;
    this.transactions = this.transactions.filter(x => x.id != id);
    return entry;
  }

  filterByCategory(category: string): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByDescription(description: string): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByBySource(source: string): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByType(type: string): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByAmountBetween(lower: number, upper: number): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByAmountAbove(amount: number): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByAmountBelow(amount: number): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByAmount(amount: number): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByDateBetween(lower: Date, upper: Date): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByDateAfter(date: Date): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByDateBefore(date: Date): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  filterByDate(date: Date): import("../models/transaction").TransactionRepositoryFilter {
    throw new Error("Method not implemented.");
  }
  collect(): Transaction[] {
    throw new Error("Method not implemented.");
  }

}
```

### Going Mongo

Time to build the repository for interfacing with the Mongo database. Start by creating a new file in the repositories directory to export a class definition that is also injectable.

**/src/repositories/transactionMongo.ts**
```typescript
import { inject, injectable } from 'inversify';
import { MongoClient, Collection, ObjectId } from 'mongodb';
import { Transaction, TransactionRepository } from '../models/transaction';
import { CONNECTIONS } from '../types';

@injectable()
export class TransactionArrayRepository implements TransactionRepository {
  public constructor(@inject(CONNECTIONS.MongoDB) client: MongoClient) {
  }
}
```

VS Code should complain because the interface needs to be implemented. I went ahead and let it add them for me and will implement each call one at a time

[Next Post][next_post]

[previous_post]: {% post_url 2020-02-29-my-web-app-journey-doing-an-oil-change %}
[next_post]: {% post_url 2020-12-06-my-web-app-journey-openfaas %}
[post_data_storev1]: {% post_url 2020-02-29-my-web-app-journey-data-store%}