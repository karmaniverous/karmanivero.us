---
# prettier-ignore
title: "What Good Looks Like: A Real-World Typescript Refactor"
excerpt: "Watch a complex Javascript configuration object collapse into declarative goodness thanks to type safety, abstraction, and generic design."
header:
  og_image: /assets/images/what-good-looks-like-banner.jpg
  teaser: /assets/images/what-good-looks-like-square.jpg
redirect_from:
  - /blog/what-good-looks-like-a-real-world-typescript-refactor/
tags:
  - design
  - entity-manager
  - projects
  - typescript
toc: true
---

<figure class="align-left drop-image">
    <img src="/assets/images/what-good-looks-like-square.jpg">
</figure>

Yesterday I posted [a bit of a screed](/blog/generic-abstractions) about iteration and the use of abstraction & generic design.

I left the ending a little ragged, as the act of writing it had focused my thoughts and opened up an avenue of approach to a solution. Which was really the point of writing the thing in the first place.

In the last 24 hours, the solution has crystallized into a radical new configuration. I'd like to share this with you, because _this_ is really the living example of all the theory I was spouting yesterday.

**Don't be afraid to read this!** It won't be that long, and I'll keep it accessible to non-developers.
{: .notice--info}

## A Scenario

As I discussed in detail yesterday, I'm working on a Typescript refactor of my [entity-manager](https://github.com/karmaniverous/entity-manager) library. This widget is intended to take most of the pain out of operating at scale with [DynamoDB](https://aws.amazon.com/dynamodb/) and other [NoSQL databases](https://en.wikipedia.org/wiki/NoSQL) in the presence of a rich data model.

To accomplish this, `entity-manager` takes a (relatively) simple configuration object as input and uses it to do all kinds of fiddly things to get the database to behave.

A picture is worth a thousand words, so let's take as our example a back-end service that manages users and their email addresses. This diagram represents those two entities and the relationship between them, along with some critical properties:

{% include figure image_path="/assets/diagrams/entity-manager-user-erd.png" caption="User service entity relationships" %}

Here's what I need from this service:

- One User can have many Emails.

- I can get a User record by `userId` and search for Users by `created`, `firstNameCanonical`, `lastNameCanonical`, `phone` & `updated`.

- I can get an Email record by `email` and search for emails by `created` & `userId`.

Moreover, since this service is currently in production, so far as the database is concerned the refactor must exactly mimic existing behavior.

## Baseline State

The `entity-manager` configuration object is everything, since the whole point of the library is to hide everything else from the developer.

So if using `entity-manager` is easy, it's because the config object is simple and straightforward. And if it's hard, it's because the config object is complex and a pain in the butt to manage.

Here's the config object for the actual production service described above. **Don't sweat the code if you aren't a developer!** I'm just showing it to you for a sense of scale.

```js
const entityConfig = {
  entities: {
    email: {
      defaultLimit: 10,
      defaultPageSize: 10,
      indexes: {
        userId: ["entityPK", "entitySK", "userId"],
      },
      keys: {
        entityPK: {
          encode: ({ entityToken = "email", shardId }) =>
            `${entityToken}!${sn2e`${shardId}`}`,
          decode: (value) =>
            value.match(/^(?<entityToken>.*)!(?<shardId>.*)$/)
              ?.groups,
        },

        entitySK: {
          encode: ({ email }) => sn2u`email#${email}`,
          decode: (value) =>
            value.match(/^email#(?<email>.*)$/)?.groups,
        },

        userId: {
          encode: ({ userId }) => userId,
          decode: (value) => ({ userId: value }),
          retain: true,
        },
      },
      sharding: {
        entityKey: ({ email }) => email,
        timestamp: ({ created }) => created,
      },
    },
    user: {
      defaultLimit: 10,
      defaultPageSize: 10,
      indexes: {
        created: ["entityPK", "entitySK", "created"],
        firstName: ["entityPK", "entitySK", "firstNameSK"],
        lastName: ["entityPK", "entitySK", "lastNameSK"],
        phone: ["entityPK", "entitySK", "phone"],
        updated: ["entityPK", "entitySK", "updated"],
      },
      keys: {
        entityPK: {
          encode: ({ entityToken = "user", shardId }) =>
            `${entityToken}!${sn2e`${shardId}`}`,
          decode: (value) =>
            value.match(/^(?<entityToken>.*)!(?<shardId>.*)$/)
              ?.groups,
        },

        entitySK: {
          encode: ({ userId }) => sn2u`userId#${userId}`,
          decode: (value) =>
            value.match(/^userId#(?<userId>.*)$/)?.groups,
        },

        created: {
          encode: ({ created }) => Number(created),
          decode: (value) => ({ created: value.toString() }),
          retain: true,
        },

        firstNameSK: {
          encode: ({ firstNameCanonical, lastNameCanonical }) =>
            sn2u`firstNameCanonical#${firstNameCanonical}${sn2e`|lastNameCanonical#${lastNameCanonical}`}`,
          decode: (value) =>
            value.match(
              /^firstNameCanonical#(?<firstNameCanonical>.*?)(?:\|lastNameCanonical#(?<lastNameCanonical>.*))?$/
            )?.groups,
        },

        lastNameSK: {
          encode: ({ lastNameCanonical, firstNameCanonical }) =>
            sn2u`lastNameCanonical#${lastNameCanonical}${sn2e`|firstNameCanonical#${firstNameCanonical}`}`,
          decode: (value) =>
            value.match(
              /^lastNameCanonical#(?<lastNameCanonical>.*?)(?:\|firstNameCanonical#(?<firstNameCanonical>.*))?$/
            )?.groups,
        },

        phone: {
          encode: ({ phone }) => phone,
          decode: (value) => ({ phone: value }),
          retain: true,
        },

        updated: {
          encode: ({ updated }) => Number(updated),
          decode: (value) => ({ updated: value.toString() }),
          retain: true,
        },
      },
      sharding: {
        entityKey: ({ userId }) => userId,
        timestamp: ({ created }) => created,
      },
    },
  },
  shardKeyToken: "shardId",
};
```

Couple of points worth noting here...

- **This thing is pretty big and complex!** To represent the two related entities & indexing requirements described [above](#a-scenario), it needed just exactly 100 lines of code.

- **This is Javascript, not Typescript!** This complex config object has to be _exactly right_, and without the [type safety](https://clouddevs.com/typescript/type-safety) Typescript provides, the developer is on his own to discover & fix any config errors. I should know: I _designed_ this thing and I screw it up regularly. 🙄

- **There's a lot of logic here!** But if you examine all those `encode` and `decode` functions, even across services, they all seem to operate the same way. That's an opportunity for abstraction.

## The New Approach

Remember, this is a Typescript refactor. That means we're going to bring typing info into the mix alongside the actual configuration object.

Here are the types that a developer would define to drive the new User service config object:

```ts
interface User {
  created: number;
  firstNameCanonical: string;
  firstNameRK: never;
  lastNameCanonical: string;
  lastNameRK: never;
  phone: string;
  updated: number;
  userId: string;
}

interface Email {
  created: number;
  email: string;
  userId: string;
}

interface EntityMap {
  user: User;
  email: Email;
}
```

The `User` & `Email` interfaces describe the entities themselves, in a way that is useful to drive `entity-manager` and also wherever else those entities show up in the code. Note that we've used the special `never` type to indicate properties the developer should not populate directly because `entity-manager` is going to generate them for us.

`EntityMap` tells `entity-manager` which parts of the configuration object will be associated with which entities.

Here's the actual config object:

```ts
export const config: Config<EntityMap, "entityPK", "entitySK"> = {
  entities: {
    user: {
      indexes: {
        created: ["entityPK", "entitySK", "created"],
        firstName: ["entityPK", "entitySK", "firstNameRK"],
        lastName: ["entityPK", "entitySK", "lastNameRK"],
        phone: ["entityPK", "entitySK", "phone"],
        updated: ["entityPK", "entitySK", "updated"],
      },
      generated: {
        firstNameRK: {
          elements: ["firstNameCanonical", "lastNameCanonical"],
        },
        lastNameRK: {
          elements: ["lastNameCanonical", "firstNameCanonical"],
        },
      },
      timestampProperty: "created",
      uniqueProperty: "userId",
    },
    email: {
      indexes: {
        userId: ["entityPK", "entitySK", "userId"],
      },
      timestampProperty: "created",
      uniqueProperty: "email",
    },
  },
  hashKey: "entityPK",
  uniqueKey: "entitySK",
};
```

If you're counting, that's just 32 lines... **a two-thirds reduction in size!** And there is _no_ persnickety functional logic, because all that stuff will now be generated by `entity-manager` based on the content of the config object.

Moreover, thanks to type safety, **it's almost impossible to screw this up!**

Typescript will alert the developer immediately if he...

- Fails to add an entity specified in the `EntityMap` interface, or adds an entity _not_ specified there.

- Specifies an entity index component that doesn't exist on the relevant entity with either a `number`, `string`, or `never` type.

- Specifies a generated property that doesn't exist on the relevant entity with a `never` type.

- Specifies a generated property component that doesn't exist on the relevant entity _without_ a `never` type.

- Specifies an entity `timestampProperty` that doesn't exist on the relevant entity or doesn't have a `number` type.

- Specifies an entity `uniqueProperty` that doesn't exist on the relevant entity or doesn't have a `number` or `string` type.

- Fails to specify exactly the right `hashKey` and `uniqueKey` properties.

- Manually populates a value on a `User` or `Email` entity _anywhere else in his code_ that `entity-manager` is supposed to generate for us.

**This, my friends, is what _good_ looks like.**

## Next Steps

There's still a lot of work to be done!

My last iteration of `entity-manager` was type-safe but assumed it was going to receive something very similar to the old config object. The job now is to add the logic that will generate the missing bits from the new config according to the rules I've discovered over two years of operating with the Javascript version of the library.

That said: the path forward from here is very clear.

Stay tuned!
