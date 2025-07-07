---
# prettier-ignore
title: "The Big Miss: Turning Lemons Into Lemonade"
excerpt: "Today was one of those days when I question my life choices. Watch me dig my way out of a hole with the same shovel I used to dig my way in."
header:
  og_image: /assets/images/lemons-banner.jpg
  teaser: /assets/images/lemons-square.jpg
redirect_from:
  - /blog/the-big-miss-turning-lemons-into-lemonade/
tags:
  - design
  - dynamodb
  - entity-manager
  - javascript
  - projects
  - troubleshooting
  - typescript
---

<figure class="align-left drop-image">
    <img src="/assets/images/lemons-square.jpg">
</figure>

Today was one of those days when I question my life choices. I could have stayed in the freaking Navy.

Tell me if you've seen this movie before: You have a tool that works well in production. You embark on a major refactor, and you do it professionally: architecture, design, documentation, and everything tested to the nines. You write a demo that doubles as an integration test, and **_it blows up like a hair-metal third-encore finale!_**

Moreover, **it invalidates one of your basic architectural premises**, and you realize with a sick jerk that the earlier version—you know, the one that _works well in production_—has been a shaky house of cards all along.

Ugh.

## The Entity Manager Refactor

[Entity Manager](/projects/entity-manager/intro/) is a library that takes the ouch out of applying the [single-table design pattern](https://aws.amazon.com/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/) to a [NoSQL](https://en.wikipedia.org/wiki/NoSQL) database. I've been running Entity Manager across 16 back-end services at [VeteranCrowd](https://veterancrowd.com) (ignore the ugly home page, please) for over a year, and it's _awesome_.

The production version of Entity Manager is written in Javascript. A few months ago I embarked on a complete rewrite of the library that encapsulates a long list of lessons learned. See [this archive](/topics/entity-manager/) for some dispatches from that journey.

The new version is cleaner, faster, and _way_ easier to configure & use. It's also Typescript from the ground up, and is about as obsessively unit tested as I can make it.

This morning I discovered that the Entity Manager refactor is a _solid_, relentlessly _perfect_ implementation of **the wrong effing thing!**

## A Hidden Bombshell

A key Entity Manager feature is behind-the-scenes declarative [partition sharding](https://aws.amazon.com/what-is/database-sharding). There are arguments for and against doing this in [DynamoDB](https://aws.amazon.com/pm/dynamodb), but Entity Manager is a platform-agnostic tool, so sharding is a feature that's been in there since the beginning.

See [this article section](/projects/entity-manager/evolving-a-nosql-db-schema/#shard-keys) for a deep dive into Entity Manager's sharding strategy, but here's the TL/DR:

- A NoSQL database record is accessed by its _primary key_.

- The primary component has two components: the _hash key_ and the _sort key_. And you can only have so much stuff with the same hash key in a single table.

- Entity Manager supports multiple entities on a single table. So it composes the hash key from the entity type and a deterministically calculated _shard key_. A typical Entity Manager hash key looks like `user!1f` or `email!2a`.

- Entity manager supports an evolving sharding scheule, so your data can scale over time. So the inputs to the shard key are the the record's unique identifier and the record creation date.

In the default case sharding is turned off, and the shard key is an empty string.

Can you see where this is going? 👀 _Because I didn't!_

The core Entity Manager package ([`entity-manager`](https://github.com/karmaniverous/entity-manager)) is all about manipulating data records to add appropriate generated properties, remove them, and support searching across shards and indexes. It is exhaustively unit-tested, but at no point does it retrieve a record from a database. At its high level of abstraction, the package doesn't even know what a database _is_.

One thing it _can_ do: generate a primary key from a sufficiently-populated entity record.

The DynamoDB-specific Entity Manager client package ([`entity-client-dynamodb`](https://github.com/karmaniverous/entity-client-dynamodb)) is all about tying the core package to the DynamoDB SDK. Given enough data—for example, the components of a primary key—it conducts CRUD operations and executes multi-index, cross-shard searches against DynamoDB. And it also works flawlessly.

But in the demo package ([`entity-manager-demo`](https://github.com/karmaniverous/entity-manager-demo)), we actually get to execute a real-world scenario:

- Create a record in a database (which returns a new unique id).

- Turn around & retrieve the record by its unique id.

- Update the record, etc...

And that's where the bomb went off.

## The Big Miss

Before I can retrieve a record by its unique id, I need to generate the record's primary key.

Internally, Entity Manager's primary key generator calculates the record's shard key... which depends on the record's creation date. **Which I don't have yet.**

So the primary key generator throws an exception and the `GET` fails.

Remember, the previous version has been running flawlessly in sixteen production services for over a year! Only the system hasn't scaled yet, so **I have sharding turned off in every service.** And, since JavaScript isn't type-safe and I wasn't validating the presence of a timestamp at runtime, the code never complained that it was getting an `undefined` instead of a `number`.

In production, Entity Manager has been returning the right answer every time... **_but only by accident!_**

Even now, this showed up as a runtime error instead of a type error because (I just checked) the `getPrimaryKey` function is [expecting the wrong type](https://github.com/karmaniverous/entity-manager/blob/main/src/EntityManager/getPrimaryKey.ts#L25), incorrect because it is too permissive. But since the refactor is _way_ more obsessive about validating runtime inputs, it caught the missing timestamp [here](https://github.com/karmaniverous/entity-manager/blob/main/src/EntityManager/updateItemHashKey.ts#L56) and threw an exception.

In cross-shard searching at scale, the relationship between shard key and record creation date provides a _huge_ advantage, since any search constrained by creation date can naturally constrain the space of shard keys it has to search. Big win.

But the most important operation anybody performs against a database is the retrieval of a single record. And in this case, Entity Manager's current sharding strategy is _fundamentally flawed_ because it won't let you retrieve a record by its unique id alone.

Damn.

## Turning Lemons Into Lemonade

In the current Entity Manager config object, a given entity's sharding strategy looks like this:

```typescript
const entityManager = new EntityManager<MyConfigMap>({
  entities: {
    email: {
      shardBumps: [
        { timestamp: 0, charBits: 1, chars: 0 },
        { timestamp: 1234567890, charBits: 2, chars: 1 },
      ],
      timestampProperty: "created",
      uniqueProperty: "email",
    },
    user: {
      shardBumps: [
        { timestamp: 0, charBits: 1, chars: 0 },
        { timestamp: 1234567890, charBits: 1, chars: 1 },
      ],
      timestampProperty: "created",
      uniqueProperty: "userId",
    },
  },
});
```

This means...

- For `email` records created before timestamp `1234567890`, all records have an empty shard key, so their hash keys are all `email!`.

- For `email` records created at and after timestamp `1234567890`, records will be distributed across a one-character, two-bit shard key. So each of these records will have one of these four hash keys: `email!0`, `email!1`, `email!2`, or `email!3`.

- `user` records have the same schedule, but their scheduled bump is to a one-bit character instead of a two-bit character. So each of these records will have one of these two hash keys: `user!0` or `user!1`.

**Obviously this is a toy scenario.** In the real-world, you'd probably want a deeper shard key. But the principle is the same.
{: .notice--info}

The calling application is responsible for providing the value of the unique property (in this case `email` or `userId`) that is the other basis (besides creation timestamp) of the shard key hash. And as we discussed above, this timestamp dependency is critical for supporting efficient cross-shard searches.

In some cases there may be no unique id intrinsic to the data. A user record is a good example of this: in our demo app we generate a [`nanoid`](https://github.com/ai/nanoid) for every new user. In other cases, the unique id is a natural part of the data, like an email address. Either way, the resulting value is encoded directly into the record's range key, like this: `userId#abc123`.

But what if we made a slight enhancement to the shard key generation strategy?

```typescript
const entityManager = new EntityManager<MyConfigMap>({
  entities: {
    email: {
      shardBumps: [
        { timestamp: 0, charBits: 1, chars: 0, rangeKeyLength: 21 },
        {
          timestamp: 1234567890,
          charBits: 2,
          chars: 1,
          rangeKeyLength: 22,
        },
      ],
      timestampProperty: "created",
      uniqueProperty: "emailId",
    },
    user: {
      shardBumps: [
        { timestamp: 0, charBits: 1, chars: 0, rangeKeyLength: 21 },
        {
          timestamp: 1234567890,
          charBits: 2,
          chars: 1,
          rangeKeyLength: 22,
        },
      ],
      timestampProperty: "created",
      uniqueProperty: "userId",
    },
  },
});
```

Either way, a record's range key is determined at the point of creation.

But now, instead of encoding the range key as described above, we can generate a random string of a specific length. This length is determined by looking up the record's creation timestamp in the config and getting `rangeKeyLength`.

Previously, the shard key generator determined the bit depth & length of the shard key by looking up some _timestamp_ in the config. This is still possible, for example when generating a shard key space to support cross shard searches.

But now, the shard key generator can _also_ determine the bit depth and length of its shard key by looking up _the length of the record's range key_ in the config!

In the User case, this means that `userId` no longer needs to be generated by the application. It's generated automatically when keys are added to the data record.

In the Email case, note that we have a new `uniqueProperty`: `emailId`. `email` is still an Email record property, but it no longer participates in key generation.

In fact, any "unique" property specific to the application (like `email`) can now be treated just like any non-unique property (like `lastName`). If you need to find a record with it, create an index and query it. Entity Manager's cross-shard queries work just as well either way.

## Conclusion

These changes are going to take a little doing to implement, but this mess isn't half the disaster it looked to be when that first integration test failed this morning.

Quite the opposite:

- We've taken unique key generation completely out of the hands of the calling application. One less dependency for sure, and also eliminated the possibility of key collisions due to some weirdness on the epplication end of things.

- We've defused a ticking time bomb that runs across the entire back end of an application I am directly responsible for. That's a career-bending problem I didn't even know I had until this morning, and it's a relief to know it will be accounted for in Entity Manager's next production release.

I'm not happy about the bug, but I'm _thrilled_ about the fix! So put it all together, and by any standard this is a win.
