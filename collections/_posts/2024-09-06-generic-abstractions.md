---
# prettier-ignore
title: "Generic Abstractions"
excerpt: "Modern software development works just like any other Hero's Journey: you've got to hoof it some ways down the road before you can see very far into the distance."
header:
  og_image: /assets/images/generic-abstractions-og.jpg
  teaser: /assets/images/generic-abstractions.png
redirect_from:
  - /blog/generic-abstractions/
tags:
  - entity-manager
  - projects
  - typescript
toc: true
---

<figure class="align-left drop-image">
    <img src="/assets/images/generic-abstractions.png">
</figure>

A few days ago I [posted](/blog/mock-db/) about the genesis of my new [`mock-db`](https://github.com/karmaniverous/mock-db) library, which mocks [DynamoDB](https://aws.amazon.com/dynamodb/)-like behavior with local JSON data.

That library is really just a bit player in a larger drama, which is the ongoing refactor from Javascript to Typescript of my [`entity-manager`](https://github.com/karmaniverous/entity-manager) library.

FYI the `entity-manager` link above takes you to the PRE-refactor branch!
{: .notice--info}

This widget provides the internal data access layer for all of my [Serverless Framework](https://www.serverless.com/) projects, and this refactor will not only make it type-safe, but will incorporate close to two years of lessons learned across well over a dozen backend services.

High stakes.

The release of `mock-db` triggered a cascade of activity, and as I look back on the past few days, I see a couple of pretty good answers to the question: **_What does a senior Typescript developer actually DO all day?_**

So I thought I'd share.

## Some Context

To understand what I've been up to, you need to know a bit about the `entity-manager` library and the problem it's meant to solve.

All of my backend projects are built on the Serverless Framework. For this discussion, the key feature is that all of these projects are _[serverless](https://en.wikipedia.org/wiki/Serverless_computing):_ they run on [AWS Lambda](https://aws.amazon.com/pm/lambda) instead of some box on a server farm. When I use a database, I use [DynamoDB](https://aws.amazon.com/pm/dynamodb), which is a [NoSQL database](https://en.wikipedia.org/wiki/NoSQL) in the AWS cloud that is also serverless.

NoSQL databases are _very_ different from traditional [relational database](https://en.wikipedia.org/wiki/Relational_database) systems (aka RDBMS) like [SQL Server](https://en.wikipedia.org/wiki/Microsoft_SQL_Server).

The _entities_ in an RDBMS (like users, orders & invoice) are each represented by a table. You tell the platform what tables you want, what data they contain, and how they relate to one another (this is the _schema_). Mostly the platform takes care of the messy parts under the hood. You can query the database however you want, and if your schema changes—say, you want to add another table or change a relationship—the platform can easily accommodate that.

That's the good news. The bad news: **all that flexibility comes at a price**. RDBMS systems are flexible and fast when you just have a few records in the system. But when you scale those records up into the millions, things start slowing down. If you aren't careful about pulling inactive data out of the system—assuming it _has_ inactive data—your fancy RDBMS can wind up unusably slow.

NoSQL databases like DynamoDB are just the opposite.

**NoSQL databases are often characterized as _schemaless_, but that isn't really accurate.** Instead, if you want your data to have any structure, it is up to _you_ to encode that structure _directly_ into your data! Rather than interacting with a sophisticated schema layer hiding a bunch of intermediate indexes, you interact _directly_ with your data and a very simple set of indexes.

So from an organizational perspective, NoSQL databases are real monsters if you want any kind of significant structure. But compared to RDBMS systems, their performance at scale is off the hook! **A well-designed NoSQL database will query 100 _million_ records just as fast as it queries 100 _thousand_.**

The problem with **_encode your schema directly into your data_** is that it isn't super obvious how to do that. Concepts like the [DynamoDB single-table design pattern](https://aws.amazon.com/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/) are great in principle, but they're really hard to implement, and they're _really_ hard to implement in a _consistent_ fashion across multiple back-end services that each use an independent DynamoDB data store.

`entity-manager` solves that problem:

- You write a simple configuration object that describes your entities, index keys, and sharding strategy.

- `entity-manager` converts that configuration into specially-formatted index properties that support all kinds of secondary indexes and sharding strategies.

- `entity-manager` provides a simple wrapper around the AWS SDK that lets you do easy things like get & update records, as well as fantastically hard things like one-shot multi-range, multi-shard, filtered secondary index queries... _without having to write a single line of DynamoDB-specific code!_

So on the one hand `entity-manager` provides an opinionated, out-of-the-box answer to the question of how to structure NoSQL data. And on the other hand, it replaces a _ton_ of code with a simple configuration that looks and works consistently across every independent microservice in your entire back end.

Pound for pound, `entity-manager` is by far the most successful piece of software I've ever written.

So if it ain't broke...

## Why Fix It?

Part of the answer can be boiled down to a single word: _Typescript_.

I picked up Typescript about a year ago after having spent about a gazillion years in the Node.js/Javascript trenches. I was late to the game, and frankly skeptical: I am a _very_ anal coder who writes _very_ tight, well-documented code, and I just didn't see the point of enforcing [type safety](https://en.wikipedia.org/wiki/Type_safety) in code that is just not very likely to suffer from type errors.

Once I actually started _using_ Typescript, I was immediately hooked.

Writing software is often very much about getting a package of data from over _here_ to over _there_, with some transformations along the way.

If you write software like a grown-up, then to get that thing done, you:

- Write down a description of the thing.

- Write some code that does the thing.

- Write some tests that prove the code you wrote actually does the thing you described.

Not strictly in that order.

Now that's a lot to keep in sync, especially when requirements change. And your requirements (or at least your understanding of them) will _always_ change. So how do you keep track? **When the various facets of your code fall out of sync, how do you actually find out?**

_That_ is what Typescript is for. When the left hand falls out of sync with the right hand, you know _instantly_. You don't have to run a test. You don't even have to _ask_. Red squiggles just appear in your IDE, and you _know_.

By an amazing kind of alchemy, the result of communicating the left hand's intent to the right hand through consistent typing is almost _always_ better code! And really _good_ Typescript is like magic: once the left hand's intent is perfectly expressed, the right hand's code just _flows_.

So these days, when I write something new, it is _always_ in Typescript. Since Serverless Framework projects are my bread and butter, and since `entity-manager` is my go-to data access layer, it just needs to be type-safe so it plays nicely with all my other Typescript goodies.

Oh, the Serverless Framework itself? They [went Typescript with version 4](https://www.serverless.com/blog/serverless-framework-v4-general-availability).

The switch to Typescript has a lot of knock-on effects. For example:

- The Javascript version relies heavily on [Lodash](https://lodash.com/), which doesn't play super well with Typescript (see [here](/blog/radash-the-new-lodash/) for more on that). So the Typescript version of `entity-manager` will use [Radash](https://github.com/sodiray/radash) instead.

- The current version of `entity-manager` uses a database [provider model](https://en.wikipedia.org/wiki/Provider_model): the library itself is generic with respect to database platform, and then there's another library that provides the DynamoDB-specific logic. Over a couple of years of development, that line got a little blurred, such that some generic functionality snuck into the DynamoDB provider, and some DynamoDB-specific functionality snuck into the implementations that consume both libraries. The Typescript refactor is a great opportunity to clean that up.

- Strong typing gives me an opportunity to apply configuration at two distinctly different levels: _type_ configurations, which directly affect developers consuming the libraries while they're writing code, and _runtime_ configurations, which affect the behavior of the library at runtime. This is a great way to improve the developer experience... and since I'm also the primary _consumer_ of these libraries, that means improving _my_ experience!

## Abstractions

A few days ago I found myself mid-swing in the `entity-manager` refactor. My strategy was to refactor the entire existing feature set and get all of the existing tests to pass before adding any new features.

I saved the hard part for last: the `query` method. This function's job is to...

- Organize a bunch of query parameters into a valid database query... possibly more than one if the query hits more than one index (_match this string against first AND last name_).

- Run the queries simultaneously across every partition shard valid for the query parameters.

- Assemble the resulting data pages into a single result set & compress the keys for the next "page" of data on each individual query down into a single blob of very high-entropy text.

This method works just fine in production, but its unit test coverage was very thin. No surprise there: the only thing that really behaves like DynamoDB is DynamoDB, and it's generally poor practice to unit test against a cloud service. But a complete refactor means I can no longer trust that production experience, so time to write better tests.

Meanwhile, remember: `entity-manager` is _generic_. Except for a little leakage, it doesn't contain _any_ DynamoDB-specific code. So I really didn't need a DynamoDB clone to test against. I just needed something that behaved _enough_ like DynamoDB to exercise the key features of the `query` method.

In practice, it needed to:

- Hold some JSON data.

- Receive a hash key, a sort key, a filter function, and a page size.

- Apply those and return a page of data with a key to the next page.

Enter [`mock-db`](/blog/mock-db/). It took about fifteen minutes of coding to understand that `mock-db` was going to be big enough and generally useful enough that it made sense to abstract it out into its own library, and then import it back into `entity-manager` as a development dependency. That's where I wound up this past Monday.

But as soon as I did that, I realized I had a new problem.

Like any other library, `mock-db` required unit tests. And, since `mock-db` was built to handle the exact same kinds of data sets as `entity-manager`, it needed the exact same set of types. Also, they both needed to perform some simple data operations like sorting and de-duping on the same kind of data.

A core principle in software engineering is _DRY:_ [_Don't Repeat Yourself_](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). So how do I use the exact same types & functions in two different libraries without repeating myself?

Easy: I created a _third_ library, called [`entity-tools`](https://github.com/karmaniverous/entity-tools), and made it a dependency of _both!_

Also, a class called `DynamoDbWrapper` encapsulates the DynamoDB-specific stuff and injects it into `EntityManager`. `DynamoDBWrapper` has a unique dependency called `aws-service-search` that handles some of that load. But in retrospect it didn't really make sense for `aws-service-search` to be a distinct dependency, so as this work proceeds I'll be merging it into `DynamoDbWrapper` when that class gets its own Typescript refactor in a few weeks.

Put it all together, and here's the before & after:

{% include figure image_path="/assets/diagrams/entity-manager-abstractions.png" caption="`entity-manager` Typescript refactor abstractions" popup=true %}

If you get the sense that I'm sort of going around in circles here, making a modest improvement at each pass: **_you're right!_** Those iterations are the essence of [Agile software development](http://127.0.0.1:4000/blog/a-modern-agile-project-manifesto/), which in the main is the only way to write software that actually works.

In fact, that's where the `entity-manager` package came from in the first place: as soon as I had more than one Serverless Framework project to manage, it became abundantly clear that they all needed to be handling data the same way. Those original projects birthed both `entity-manager` and `DynamoDbWrapper`.

So the arrows in the diagram above don't just represent the dependency flow of these components. They also represent the historical decomposition of the work.

## Genericization

Is that an awful word, or what?

As soon as Typescript enters the picture—stay with me here—everything gets _typed_.

From a trivial perspective, this means that Typescript will complain if I try to do something dumb like treat a string as an integer. I can kind of cheat, accidentally or on purpose, by using the special-purpose `any` type, which means exactly what it implies. But if I'm smart, I'll set Typescript up to complain when I do _that_, too. Which leaves me no alternative but to type consistently.

But what does that actually _mean_?

Say I have a simple type representing a user, which I want to manage in my data store with `entity-manager`. I can define that type like this:

```ts
interface User {
  id: number;
  name: string;
  optional?: string;
  data: JsonData; // <-- what is this??
}
```

Now say I want to create a new instance of `entity-manager`. I can write this:

```ts
const entityManager = new EntityManager(config);
```

... where `config` is the configuration I use to describe my entities, indexes, and sharding strategy. But hang on... how do I know if my configuration is valid?

For example:

- My database is supposed to handle `User` and `Email` entities, but in my configuration I have a `Foo` entity.

- DynamoDB can't handle the `BigInt` data type, but I have a `BigInt` field in my `User` entity.

- DynamoDB can only build indexes on scalar types like `number` and `string`, but I tried to create an index on the `data` property.

Remember, in Typescript we have two kinds of configuration:

- _Runtime_ configuration operates on deployed code and tries to catch my mistakes before they can do any damage.

- _Type_ configuration operates in my development environment to prevent me from making mistakes in the first place.

Guess which kind is preferable.

So when I create a new instance of `EntityManager`, Typescript style, I should be able to tell it:

- What entity types it will be expected to manage. _Because obviously it won't always be the same one._

- What building blocks are allowed to comprise those entities. _Because I won't always be working in DynamoDB._

- Which of those building blocks are allowed to be part of an index. _Same reason._

If I do that right, a couple of things will happen:

- Typescript will catch most of my errors before I ever deploy my code.

- When I write that configuration object for a specific implementation (say, `User`), Typescript will know in advance what to expect. If I make a mistake, I'll get a red squiggle. But at least as important, once I start typing, Typescript will use those expectations to provide me with [_intellisense_ and _autocomplete_](https://code.visualstudio.com/docs/editor/intellisense): it will anticipate what I am trying to do, put relevant instructions at my fingertips, and finish my typing for me.

That's worth doing a little work for.

Let's start with data types allowed by the database. Say I define this type:

```ts
type DefaultProperty =
  | string
  | number
  | boolean
  | null
  | undefined
  | { [key: string]: DefaultProperty } // JSON objects
  | DefaultProperty[]; // JSON arrays
```

This recursive type definition describes a bunch of scalar types, as well as JSON objects and arrays. Now I can alter my `EntityManager` class definition so it looks like this:

```ts
export class EntityManager<P = DefaultProperty> { ... }
```

Say my database can't handle structured data like JSON objects and arrays. I can define an appropriate new property type and create a new instance of `EntityManager` like this:

```ts
type CustomProperty = string | number | boolean | null | undefined;

const entityManager = new EntityManager<CustomProperty>(config);
```

What you are looking at there is a _generic_ class definition. The `EntityManager` class is _parameterized_ by a type `P`, which defaults to `DefaultProperty`. When I create a new instance of `EntityManager`, I can _specify_ the type `P` by passing it as a _type argument_ to the class constructor... and, just like magic, Typescript will yell at me if I write code that wouldn't handle this type properly!

Take this to its logical conclusion, and when you see the full `EntityManager` Typescript refactor, creating a new instance will look something like this:

```ts
interface User extends Entity {
  userId: number;
  name: string;
  optional?: string;
  data: JsonData;
}

interface Email extends Entity {
  email: string;
  userId: number;
}

interface MyEntityMap extends EntityMap {
  user: User;
  email: Email;
}
```

Then I'll be able to write a config object & define a new instance of `EntityManager` like this:

```ts
const config: EntityManagerConfig<MyEntityMap> = {
  entities: {
    user: {
      primaryKey: ['userId'],
      indexes: {
        name: { 'name' },
        email: { 'email' },
      },
    },
    email: {
      primaryKey: ['email'],
      indexes: {
        userId: { hashKey: 'userId' },
      },
    },
  },
};

const entityManager = new EntityManager(config);
```

... and internally the types will just _work_. The `MyEntityMap` type will impose design-time constraints on the `config` object and the rest of my code, and prevent me (and other users of `EntityManager`) from making expensive mistakes while coding.

## What... That's It?

Seems like kind of a ragged ending to a long article, doesn't it. Know why?

Because **the very act of writing this article sent me into a couple of new directions** that will significantly (and positively!) affect the outcome for `EntityManager`.

Remember the question at the top of the article? _What the hell do I actually_ do _with my time?_

Well... _this._

As I mentioned above, iteration is the heart & soul of modern software development. It works just like any other [Hero's Journey](https://en.wikipedia.org/wiki/Hero%27s_journey): you've got to hoof it some ways down the road before you can see very far into the distance.

More to come. Meanwhile: **_time to iterate!_**
