---
title: "Entity Manager: A Demonstration"
excerpt: Presenting a step-by-step Typescript implementation of a realistic data model against DynamoDB, with the help of Entity Manager.
permalink: /projects/entity-manager/demo/
header:
  og_image: /assets/collections/entity-manager/configuration-banner.jpg
  overlay_image: /assets/collections/entity-manager/configuration-banner-half.jpg
  teaser: /assets/collections/entity-manager/configuration-square.jpg
under_construction: true
related: true
tags:
  - aws
  - dynamodb
  - entity-manager
  - logging
  - nosql
  - projects
  - typescript
---

<figure class="align-left drop-image">
    <img src="/assets/collections/entity-manager/configuration-square.jpg">
</figure>

Your project's data needs are complex. **Entity Manager** simplifies the problem by encapsulating a provider-agnostic, highly opinionated approach to the single-table design pattern.

At the end of the day, though, you have to implement a _specific_ data model against a _specific_ database platform. This guide presents a step-by-step Typescript implementation of a realistic data model against [DynamoDB](https://aws.amazon.com/dynamodb/), with the help of **Entity Manager**.

The [`entity-manager-demo`](https://github.com/karmaniverous/entity-manager-demo) repository contains the full implementation documented below.

**This page is under construction!** The Typescript refactor is nearly complete, and I'm busy building the demo & syncing up this documentation. Please check back soon for updates and [drop me a note](https://github.com/karmaniverous/entity-manager/discussions) with any questions or ideas!.
{: .notice--warning}

## An Overview

As the basis of this demonstration we will use the same data model, table design, and index structure we worked up in [Evolving a NoSQL DB Schema](/projects/entity-manager/evolving-a-nosql-db-schema/). If you haven't read this article yet, I recommend you do so before proceeding as it will help you understand _why_ we settled on the design we chose.

If you just want to review the resulting design, see the [Recap](/projects/entity-manager/evolving-a-nosql-db-schema/#recap) section at the end of that article.

{% include figure image_path="/assets/diagrams/entity-manager-evolving-a-nosql-db-schema-data-model.png" caption="_User service data model._" %}

While all **Entity Manager** entity records are technically sharded, by default each record's shard key is an empty string, resulting in effectively unsharded data. To demonstrate **Entity Manager**'s ability to scale, we will configure a sharding schedule for each entity that will allow us to test both unsharded and sharded scenarios.

The details of an API implementation are beyond the scope of this demo, but we will develop the handler functions an API would call to perform the following operations against the above data model in DynamoDB using **Entity Manager**:

- Email entity:
  - Create a new email record.
  - Delete an email record.
  - Retrieve a list of email records by `userId`.
- User entity:
  - Create a new user record.
  - Retrieve a user record by `userId`.
  - Update a user record.
  - Delete a user record and associated email records.
  - Retrieve a list of user records by various search criteria, including a multi-index match against both `firstName` and `lastName`.

We'll also demonstrate how to leverage your **Entity Manager** config to generate a definition for your DynamoDB table, and how to use the [`entity-client-dynamodb`](https://github.com/karmaniverous/entity-client-dynamodb) to create this table and efficiently perform other table-level database operations in DynamoDB.

We'll write [`mocha`](https://mochajs.org)/[`chai`](https://www.chaijs.com/) unit tests to exercise all this functionality against DynamoDB using the [DynamoDB Local](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html) emulator running in a Docker container. This will allow you to run the demo on your local machine without incurring any AWS costs (though these would be trivial) and without requiring an active connection to AWS.

You should be abe to pull the repository, install dependencies, and run all unit tests successfully within just a few minutes. **So let's get started!**

For convenience, this repository uses my [Typescript NPM Package Template](https://github.com/karmaniverous/npm-package-template-ts), only I've stripped out the CLI & [Rollup](https://rollupjs.org/) build and have disabled NPM publishing. So what remains is a pure, [semantic-versioned](https://semver.org/) Typescript "package" with a bunch of unit tests: perfect for a demo that can evolve over time, useless for anything else.
{: .notice--info}

## DynamoDB Local Integration

There are a couple of different ways to run DynamoDB locally. See the [DynamoDB Local](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html) documentation for details.

To keep things simple, this demo executes its tests against DynamoDB running in a Docker image. The only requirement is that [Docker Desktop](https://www.docker.com/products/docker-desktop/) be installed & running. When you execute a test, the test suite will download the Docker image if it isn't already present, start the container, run the tests, and then stop & delete the container.

Setting up Docker Desktop is beyond the scope of this guide, but if you're on a Windows machine and have [Chocolatey](https://community.chocolatey.org/), it's easy: run `choco install docker-desktop` from an admin prompt. You'll want to restart your machine once installation completes.

**If your first test execution seems to hang, check the output panel!** The DynamoDB Local Docker image takes a couple of minutes to download & install, but you'll only have to do that once. If your tests fail outright, make sure Docker Desktop is actually running!
{: .notice--warning}

## Setting Up The Demo

After you have Docker Desktop installed & running, follow these steps:

1. Clone the [`entity-manager-demo`](https://github.com/karmaniverous/entity-manager-demo) repository to your local machine.

1. Install dependencies by running `npm install` from the repository root.

1. Optionally, install [Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter) to make it a bit easier to follow along with the examples below.

That's it! Check your work by running `npm test` from the repository root. If all the tests pass, you're ready to start exploring the code!

**If you run into any trouble**, please [start a discussion](https://github.com/karmaniverous/entity-manager-demo/discussions) and I'll help!
{: .notice--info}

## Logger

{% include figure image_path="https://raw.githubusercontent.com/karmaniverous/entity-manager-demo/main/assets/logger.png" caption="_Logger configuration._" %}

All packages in the **Entity Manager** ecosystem perform extensive debug logging by default. This is often very useful when troubleshooting projects that leverage these packages, but it can also inject a lot of noise into the console when you're trying to focus on your _own_ code with its _own_ debug logging.

**Entity Manager** packages also support an injected logger object. To address the noise issue, we will...

- Alias `console` to `logger` and use it everywhere we want logging in our demo. The demo will work just as well if you replace `console` with `winston` or some other logger of choice.

- Use the `controlledProxy` function to proxy the `logger` object and disable the `debug` endpoint. When we inject the resulting `errorLogger` object into our `EntityManager` and `Entity Client` instances, all of their internal debug logging will be suppressed.

Visit [`logger.ts`](https://github.com/karmaniverous/entity-manager-demo/blob/main/src/logger.ts) to see this code in context.

```ts
import { controlledProxy } from "@karmaniverous/controlled-proxy";

// Use the console logger. This could easily be replaced with a
// custom logger like winston.
export const logger = console;

// Proxy the logger & disable debug logging.
export const errorLogger = controlledProxy({
  defaultControls: { debug: false },
  target: logger,
});
```

## `EntityManager` Configuration

{% include figure image_path="https://raw.githubusercontent.com/karmaniverous/entity-manager-demo/main/assets/entityManager.png" caption="_`EntityManager` configuration._" %}

**Entity Manager** configuration is a complex topic! Our purpose here is to focus specifically on our demonstration scenario. [Click here](/projects/entity-manager/configuration/) for a deeper dive into all aspects of **Entity Manager** configuration.

**Entity Manager is a Typescript-first tool!** If you are writing Javascript, you can skip the type-related parts of this guide and your config will still be validated for you at run time. You just won't get the compile-time type checking that Typescript provides.
{: .notice--warning}

The [`EntityManager`](https://docs.karmanivero.us/entity-manager/classes/index.EntityManager.html) class configuration object is defined in the [`Config`](https://docs.karmanivero.us/entity-manager/types/entity_manager.Config.html) type. Internally this type is _very_ complex, in order to support all kinds of compile-time validations. Once the config is parsed, though, it takes on the [`ParsedConfig`](https://docs.karmanivero.us/entity-manager/types/entity_manager.ParsedConfig.html) type, **which has the same form and is quite a bit easier to read!**

Creating an `EntityManager` instance is really about creating a valid `Config` object.

The `Config` type has four type parameters. Only the first one is required, and for this demo we will use defaults for the other three. Here they are:

- `M extends`[`EntityMap`](/projects/entity-manager/configuration/#the-entitymap-type) - This is the most important parameter and will be different in every implementation. We'll address it below.

- `HashKey extends string` - This is the name of the generated hash key property that will be shared across all entities in the configuration. The default value is `'hashKey'`, and we will use that here.

- `RangeKey extends string` - This is the name of the generated range key property that will be shared across all entities in the configuration. The default value is `'rangeKey'`, and we will use that here.

- `T extends`[`TranscodeMap`](/projects/entity-manager/configuration/#the-transcodemap-type) - Relates the name of a [transcode](/projects/entity-manager/configuration/#transcodes) to the type of the value being transcoded. This parameter defaults to an extensible [DefaultTranscodeMap](https://docs.karmanivero.us/entity-tools/interfaces/entity_tools.DefaultTranscodeMap.html) type that will serve our purposes here.

The `EntityManager` constructor also takes a `logger` argument. To minimize noise in the demo console, the injected `errorLogger` proxies `console` to disable debug logging in the `EntityManager` instance while keeping it enabled elsewhere. See my [`controlled-proxy`](https://github.com/karmaniverous/controlled-proxy) repo for more info!

Let's compose our configuration's `EntityMap`.

### `MyEntityMap` Type

An [`EntityMap`](http://localhost:4000/projects/entity-manager/configuration/#the-entitymap-type) is just a map of Typescript interfaces that define the structure of each entity in the configuration. The keys of the map are the entity names, and the values are the entity interfaces.

As a special convention, within each interface we identify _generated properties_ (the ones marked with a ⚙️ in our [table design](/projects/entity-manager/evolving-a-nosql-db-schema/#table-properties)) with a `never` type. This is a signal to the `Config` type that these properties require special support & configuration. See the `EmailEntity` and `UserEntity` interfaces below for examples.

While we are at it, we will also construct and export the `Email` and `User` types. These are the same as `EmailEntity` and `UserEntity`, respectively, but we've stripped out all properties with the `never` type. These are useful for the rest of our application code, which doesn't need to know about properties that are specific to data operations.

Here is the definition of `MyEntityMap` from [`entityManager.ts`](https://github.com/karmaniverous/entity-manager-demo/blob/main/src/entityManager.ts):

```ts
import type { EntityMap } from "@karmaniverous/entity-manager";
import type {
  Entity,
  PropertiesNotOfType,
} from "@karmaniverous/entity-tools";

// Email entity interface. never types indicate generated properties.
interface EmailEntity extends Entity {
  created: number;
  email: string;
  userHashKey: never; // generated
  userId: string;
}

// Email type for use outside data operations.
export type Email = Pick<
  EmailEntity,
  PropertiesNotOfType<EmailEntity, never>
>;

// User entity interface. never types indicate generated properties.
interface UserEntity extends Entity {
  beneficiaryId: string;
  created: number;
  firstName: string;
  firstNameCanonical: string;
  firstNameRangeKey: never; // generated
  lastName: string;
  lastNameCanonical: string;
  lastNameRangeKey: never; // generated
  phone?: string;
  updated: number;
  userBeneficiaryHashKey: never; // generated
  userHashKey: never; // generated
  userId: string;
}

// Email type for use outside data operations.
export type User = Pick<
  UserEntity,
  PropertiesNotOfType<UserEntity, never>
>;

// Entity interfaces combined into EntityMap.
interface MyEntityMap extends EntityMap {
  email: EmailEntity;
  user: UserEntity;
}
```

### `config` Object

The `config` object has a lot of moving parts, so it helps to come at the problem from a specific direction.

The properties of the demo config object defined below are arranged accordingly and explained in the comments. For the mopart, only the Email entity config is commented, since the User config follows the same pattern.

Here are some important references from the comments:

- The demo [table design](/projects/entity-manager/evolving-a-nosql-db-schema/#table-properties).

- The demo [index design](http://localhost:4000/projects/entity-manager/evolving-a-nosql-db-schema/#indexes).

- [Transcodes](/projects/entity-manager/configuration/#transcodes) perform and reverse the conversion of a value into a string, often at a fixed width, for inclusion in a generated property while preserving its sorting characteristics. The [`defaultTranscodes`](https://github.com/karmaniverous/entity-tools/blob/main/src/defaultTranscodes.ts) object is used here since a `transcodes` object is not defined in the config. Available transcodes are:
  - `bigint` - Renders a `bigint` as a variable-width string.
  - `bigint20` - Pads a `bigint` to a fixed width of 20 characters.
  - `boolean` - Renders as `'t'` or `'f'`.
  - `fix6` - Pads a number to a fixed width with 6 decimal digits.
  - `int` - Pads a signed integer to a fixed width.
  - `number` - Renders a `number` as a variable-width string.
  - `string` - Variable-length pass-through.
  - `timestamp` - Pads a UNIX ms timestamp to a fixed width.

Visit [`entityManager.ts`](https://github.com/karmaniverous/entity-manager-demo/blob/main/src/entityManager.ts) to see this code in context.

```ts
import {
  type Config,
  EntityManager,
} from "@karmaniverous/entity-manager";

import { errorLogger } from "./logger";

// Current timestamp will act as break point for sharding schedule.
const now = Date.now();

// Config object for EntityManager.
// Using default values for HashKey, RangeKey, and TranscodeMap
// type params.
const config: Config<MyEntityMap> = {
  // Common hash & range key properties for all entities. Must
  // exactly match HashKey & RangeKey type params.
  hashKey: "hashKey",
  rangeKey: "rangeKey",

  // Entity-specific configs. Keys must exactly match those of
  // MyEntityMap.
  entities: {
    // Email entity config.
    email: {
      // Source property for the Email entity's hash key.
      uniqueProperty: "email",

      // Source property for timestamp used to calculate Email
      // shard key.
      timestampProperty: "created",

      // Email entity's shard bump schedule. Records created before
      // now are unsharded (1 possible shard key). Records created
      // after now have a 1-char, 2-bit shard key (4 possible shard
      // keys).
      shardBumps: [{ timestamp: now, charBits: 2, chars: 1 }],

      // Email entity generated properties. These keys must match
      // the ones with never types in the Email interface defined
      // above, and are marked with a ⚙️ in the table design.
      generated: {
        userHashKey: {
          // When true, if any element is undefined or null, the
          // generated property will be undefined. When false,
          // undefined or null elements will be rendered as an
          // empty string.
          atomic: true,

          // Elements of the generated property. These MUST be
          // ungenerated properties (i.e. not marked with never
          // in the Email interface) and MUST be included in the
          // entityTranscodes object below. Elements are applied
          // in order.
          elements: ["userId"],

          // When this value is true, the generated property will
          // be sharded.
          sharded: true,
        },
      },

      // Indexes for the Email entity as specified in the index
      // design.
      indexes: {
        // An index hashKey must be either the global hash key or a
        // sharded generated property. Its rangeKey must be either
        // the global range key, an scalar ungenerated property, or
        // an unsharded generated property. Any ungenerated
        // properties used MUST be included in the entityTranscodes
        // object below.
        created: { hashKey: "hashKey", rangeKey: "created" },
        userCreated: {
          hashKey: "userHashKey",
          rangeKey: "created",
        },
      },

      // Transcodes for ungenerated properties used as generated
      // property elements or index components. Transcode values
      // must be valid config transcodes object keys. Since this
      // config does not define a transcodes object it uses
      // defaultTranscodes exported by @karmaniverous/entity-tools.
      elementTranscodes: {
        created: "timestamp",
        userId: "string",
      },
    },
    // User entity config.
    user: {
      uniqueProperty: "userId",
      timestampProperty: "created",
      shardBumps: [{ timestamp: now, charBits: 2, chars: 1 }],
      generated: {
        firstNameRangeKey: {
          atomic: true,
          elements: [
            "firstNameCanonical",
            "lastNameCanonical",
            "created",
          ],
        },
        lastNameRangeKey: {
          atomic: true,
          elements: [
            "lastNameCanonical",
            "firstNameCanonical",
            "created",
          ],
        },
        userBeneficiaryHashKey: {
          atomic: true,
          elements: ["beneficiaryId"],
          sharded: true,
        },
        userHashKey: {
          atomic: true,
          elements: ["userId"],
          sharded: true,
        },
      },
      indexes: {
        created: ["hashKey", "rangeKey", "created"],
        firstName: ["hashKey", "rangeKey", "firstNameRangeKey"],
        lastName: ["hashKey", "rangeKey", "lastNameRangeKey"],
        phone: ["hashKey", "rangeKey", "phone"],
        updated: ["hashKey", "rangeKey", "updated"],
        userBeneficiaryCreated: [
          "hashKey",
          "rangeKey",
          "userBeneficiaryHashKey",
          "created",
        ],
        userBeneficiaryFirstName: [
          "hashKey",
          "rangeKey",
          "userBeneficiaryHashKey",
          "firstNameRangeKey",
        ],
        userBeneficiaryLastName: [
          "hashKey",
          "rangeKey",
          "userBeneficiaryHashKey",
          "lastNameRangeKey",
        ],
        userBeneficiaryPhone: [
          "hashKey",
          "rangeKey",
          "userBeneficiaryHashKey",
          "phone",
        ],
        userBeneficiaryUpdated: [
          "hashKey",
          "rangeKey",
          "userBeneficiaryHashKey",
          "updated",
        ],
      },
      elementTranscodes: {
        beneficiaryId: "string",
        created: "timestamp",
        firstNameCanonical: "string",
        lastNameCanonical: "string",
        phone: "string",
        updated: "timestamp",
        userId: "string",
      },
    },
  },
};

// Configure & export EntityManager instance.
export const entityManager = new EntityManager(config, errorLogger);
```

### Item Types

The `Email` and `User` interfaces defined above are not useful for database operations because...

- they don't include the `hashKey` and `rangeKey` properties that we specified in our **Entity Manager** `config` object, and

- their generated properties all have a `never` type.

The [`ItemMap`](http://localhost:4000/projects/entity-manager/configuration/#the-itemmap-type) type takes our `EntityMap` type as a parameter and returns a map of the correct item types. Let's do this and export the result for use in our endpoint handlers.

Visit [`entityManager.ts`](https://github.com/karmaniverous/entity-manager-demo/blob/main/src/entityManager.ts) to see this code in context.

```ts
import type { ItemMap } from "@karmaniverous/entity-manager";

// Construct ItemMap type from MyEntityMap.
type MyItemMap = ItemMap<MyEntityMap>;

// Export EmailItem & UserItem types for use in other modules.
export type EmailItem = MyItemMap["email"];
export type UserItem = MyItemMap["user"];
```

## `EntityClient` Configuration

{% include figure image_path="https://raw.githubusercontent.com/karmaniverous/entity-manager-demo/main/assets/entityClient.png" caption="_`EntityClient` configuration._" %}

The [`EntityClient`](https://docs.karmanivero.us/entity-client-dynamodb/classes/index.EntityClient.html) class combines the [`DynamoDBClient`](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/dynamodb/) and [`DynamoDBDocument`](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-lib-dynamodb/Class/DynamoDBDocument/) classes from the AWS SDK with some higher-level functions to provide a simplified interface over key interactions with a DynamoDB database, as well as improved handling of batch operations.

For example:

- The [`createTable`](https://docs.karmanivero.us/entity-client-dynamodb/classes/index.EntityClient.html#createTable) method leverages its internal `DynamoDBClient` instance to create a table, then calls [`waitUntilTableExists`](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-client-dynamodb/Variable/waitUntilTableExists) to block further execution until the new table is actually available for data operations.

- The [`putItems`](https://docs.karmanivero.us/entity-client-dynamodb/classes/index.EntityClient.html#putItems) method breaks an array of entity items into multiple batches, then leverages the `DynamoDBDocument` [`BatchWriteCommand`](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-lib-dynamodb/Class/BatchWriteCommand/) to perform throttled batched writes to the database in parallel.

The class also provides direct access to the underlying `DynamoDBClient` and `DynamoDBDocument` instances, so any operations not supported by enhanced `EntityClient` methods are also available. This demo will provide examples of both modes of operation.

See the [`entity-client-dynamodb`](https://github.com/karmaniverous/entity-client-dynamodb) repository for more info.

As with the [`EntityManager` instance configuration](#entitymanager-configuration) above, we have injected an `errorLogger` object that proxies `console` to disable debug logging in the `EntityClient` instance while keeping it enabled elsewhere. See my [`controlled-proxy`](https://github.com/karmaniverous/controlled-proxy) repo for more info!

Otherwise, the [`EntityClientOptions`](https://docs.karmanivero.us/entity-client-dynamodb/interfaces/index.EntityClientOptions.html) type is only a slight extension of the familiar [`DynamoDBClientConfig`](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-client-dynamodb/Interface/DynamoDBClientConfig) type, so configuring the `EntityClient` instance for this demo is straightforward.

Visit [`entityClient.ts`](https://github.com/karmaniverous/entity-manager-demo/blob/main/src/entityClient.ts) to see this code in context.

```ts
import { EntityClient } from "@karmaniverous/entity-client-dynamodb";

import { errorLogger } from "./logger";

export const entityClient = new EntityClient({
  credentials: {
    accessKeyId: "fakeAccessKeyId",
    secretAccessKey: "fakeSecretAccessKey",
  },
  endpoint: "http://localhost:8000",
  logger: errorLogger,
  region: "local",
});
```

## Endpoint Handlers

{% include figure image_path="https://raw.githubusercontent.com/karmaniverous/entity-manager-demo/main/assets/handler.png" caption="_CRUD & search handler structure._" %}

Every handler function defined below follows the pattern illustrated in the diagram above. Here's a breakdown of the key elements in the diagram:

- **`User`** is the type used in your application to represent a User object, not including all the generated properties used by **Entity Manager** to support database operations. Depending on handler requirements, `Email` is also available.

- **`params`** are the parameters received by the `handler` function. In most cases this object's type will be some variant on the `User` or `Email` type.

- **`UserItem`** is the complete User type stored in the database, including all generated properties specified in the `EntityManager` [configuration](https://docs.karmanivero.us/entity-manager/types/entity_manager.ParsedConfig.html). Depending on handler requirements, `EmailItem` is also available.

- **`handler`** is the function that handles the actual data request. As we will see below, with **Entity Manager** in place this code can be _very_ compact and efficient! Internally, the `handler` function will use the `UserItem` and `EmailItem` types to interact with the database.

- **`entityManager`** is an instance of the [`EntityManager`](https://docs.karmanivero.us/entity-manager/classes/index.EntityManager.html) class, which is initialized with a configuration reflecting the design summarized [here](/projects/entity-manager/evolving-a-nosql-db-schema/#recap). This object gives our handler the ability to add and remove generated properties from a `UserItem` or `EmailItem` and to perform query and CRUD operations on these entities in the database.

- **`entityClient`** is an instance of the DynamoDB-specific [`EntityClient`](https://docs.karmanivero.us/entity-client-dynamodb/classes/index.EntityClient.html) class, initialized to connect with the User Service table in DynamoDB. In principle we could accomplish everything in this demo using [`DynamoDBClient`](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/dynamodb/) and [`DynamoDBDocument`](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-lib-dynamodb/Class/DynamoDBDocument/) from the native DynamoDB SDK, but the `EntityClient` class simplifies database interactions and eliminates a lot of noise that would otherwise interfere with the clarity of this demo. We'll also use a utility function from the same library to generate our DynamoDB table definition from our **Entity Manager** config!

- **`shardQueryMapBuilder`** is a [`ShardQueryMapBuilder`](https://docs.karmanivero.us/entity-client-dynamodb/classes/index.QueryBuilder.html) instance declared internally by handler functions (like search endpoints) that require the ability to perform cross-shard, multi-index database queries. Each instance is handler-specific, so this object is not shared between handlers.

- **`logger`** is simply an alias of `console`. Feel free to replace it with your favorite logger!

### Email Entity

{% include figure image_path="https://raw.githubusercontent.com/karmaniverous/entity-manager-demo/main/assets/email-types.png" caption="_Email entity types._" %}

#### Create Email

TODO

#### Read Email

TODO

#### Delete Email

TODO

#### Search Emails

TODO

### User Entity

{% include figure image_path="https://raw.githubusercontent.com/karmaniverous/entity-manager-demo/main/assets/user-types.png" caption="_User entity types._" %}

#### Create User

TODO

#### Read User

TODO

#### Update User

TODO

#### Delete User

TODO

#### Search Users

TODO
