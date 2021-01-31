---
layout: post
title: My Web App Journey - Data Store
date: 2020-02-29 14:32:01.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- General
tags:
- Data Repository
- MongoDB
- Scope Creep
author: Jonathan Tweedle
permalink: "/2020/02/29/my-web-app-journey-data-store/"
---
In the [previous chapter][previous_post] my initial plan was to improve the data repository to leverage MongoDB. I ended up deviating from that plan to focus on some SOLID principles.

This chapter will be about introducing MongoDB connectivity to  manage the data persistence of the API.

## First Steps

I start by opening my project and going to the terminal window. Making sure that I am currently at the root folder of my project, install the MongoDB dependencies to the project.

```
npm install MongoDB
```

This downloads the required dependency files, while updating the project configuration file (package.json) as needed.

## A new world

I want to add a new repository that uses a connection to the MongoDB server. The recommended design is to only create a single instance of the client that is reused instead of creating a new connection for every call. Sticking to the SOLID principles I am going to inject this into the repository. 

To start I need to define a type that will hold the symbols used for the injection pattern.  This will update the types as follows:

**/src/types.ts**
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

export const CONNECTIONS = {
  MongoDB: Symbol.for('MongoDB')
};
```

## The wrong path

Sometimes we don't realize some choices are not the right ones. Just as I was about to get my new repository in place, I uncovered a flaw in my interface design.

It is too generic. 

I was trying to rely on the freedom to move the ability to filter matches away from the repository by relying on predicates. This conflicted with the MongoDB driver which uses a query language.

I need to update my interface to introduce the various methods by which I want to interact with the data. I decided to add some freedom by using regular expressions to allow multiple matches if needed.

**/src/models/transaction.ts**
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

  /** add a new transaction to the repository.
   * @returns transaction. null if it failed to add
   */
  add(record: Transaction): Transaction;

  /** updates a transaction in the repository.
   * @returns transaction. null if unsuccessful */
  update(record: Transaction): Transaction;

  /** removes a record from the repository.
   * @returns transaction. null if unsuccessful */
  remove(record: Transaction): Transaction;

  /** get all the transactions in the repository.
   * @returns array of transaction. null if no records found
   */
  all(): Array<Transaction>;

    /** finds the first matching Transaction in the repository based on its ID. supports regular expression matching.
   * @returns transaction. null if not found
   */
  getByID(id: string): Transaction;

  /** find transactions in a category. supports regular expression matching.
   * @returns array of transaction. null if no records found
  */
  findByCategory(category: string): Array<Transaction>;

  /** find transactions based on the description. supports regular expression matching.
   * @returns array of transaction. null if no records found
  */
  findByDescription(description: string): Array<Transaction>;

  /** find transactions based on the source. supports regular expression matching.
   * @returns array of transaction. null if no records found
  */
  findByBySource(source: string): Array<Transaction>;

  /** find transactions based on the type. supports regular expression matching.
   * @returns array of transaction. null if no records found
  */
  findByType(type: string): Array<Transaction>;

  /** find transactions between two given amounts (inclusive).
   * @returns array of transaction. null if no records found
  */
  findByAmountBetween(lower: number, upper: number): Array<Transaction>;

  /** find transactions above a given amount (exclusive).
   * @returns array of transaction. null if no records found
  */
  findByAmountAbove(amount: number): Array<Transaction>;

  /** find transactions below a given amount (exclusive).
   * @returns array of transaction. null if no records found
  */
  findByAmountBelow(amount: number): Array<Transaction>;

  /** find transactions at a given amount (inclusive).
   * @returns array of transaction. null if no records found
  */
  findByAmount(amount: number): Array<Transaction>;

  /** find transactions between two given dates (inclusive).
   * @returns array of transaction. null if no records found
  */
 findByDateBetween(lower: Date, upper: Date): Array<Transaction>;

 /** find transactions after a given date (exclusive).
  * @returns array of transaction. null if no records found
 */
 findByDateAfter(date: Date): Array<Transaction>;

 /** find transactions before a given amounts (exclusive).
  * @returns array of transaction. null if no records found
 */
 findByAmountBefore(date: Date): Array<Transaction>;

 /** find transactions on a given amounts.
  * @returns array of transaction. null if no records found
 */
 findByDate(date: Date): Array<Transaction>;
}
```

### When Life Happens

So half way through this step I got started a new course, moved halfway around the world, and well got completely sidetracked. 

In the interim angular has gone from version 7.0.3 up to 9.0. We now have the Raspberry Pi 4 which can is recently fully supported through the latest kernel. NodeJs has gone from version v8.12 up to v12.16. 

I tried to startup the project which regrettably did not work, initial analysis is linked to my decision to diverge from comparator parameters with highly generic methods to more specific methods. It turns out I still had some changes to make to update my code to use the new interface I had introduced.

This however introduces a typical dilemma when working on a project over such a long time frame. What if you have updated your local environment which means you will need to first revert your development system to match your project and then make the changes or do you bite the bullet and upgrade your project to bring it to the current version in play?

I plan to follow the latter course and update all the elements of the project to use the latest versions. I will hopefully be able to deal with the fallout when i deploy to my hosting system (when it arrives or i build a new one ... OpenFaaS pi cluster maybe?) .

So that will conclude this chapter to move on the [next chapter][next_post] which will focus on updating everything.

### Contributing Research Material

<span style="font-size:12px;"><a href="https://fullstack-developer.academy/sharing-a-mongodb-connection-in-node-js-express/">Piros, Tamas. Sharing a MongoDB connection in NodeJS/Express. Web Article. 13 September 2018.</a></span>

<span style="font-size:12px;"><a href="https://www.youtube.com/watch?v=rtXpYpZdOzM&amp;feature=youtu.be">Hamedani, Mosh. Repository Pattern with C# and Entity Framework, Done Right. Youtube. 15 October 2015.</a></span>

[previous_post]: {% post_url 2019-01-27-my-web-journey-data-repository %}
[next_post]: {% post_url 2020-02-29-my-web-app-journey-doing-an-oil-change %}