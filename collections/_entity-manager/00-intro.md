---
title: Meet Entity Manager!
excerpt: NoSQL shifts the burden of complexity from the database platform to the developer. **Entity Manager** sweeps it under the rug!
permalink: /projects/entity-manager/intro/
redirect_from:
  - /projects/entity-manager/
related: true
tags:
  - aws
  - dynamodb
  - entity-manager
  - nosql
  - projects
  - typescript
---

<figure class="align-left drop-image">
    <img src="/assets/collections/entity-manager/entity-manager-square.jpg">
</figure>

The **Entity Manager** package implements rational indexing & cross-shard querying at scale in your [NoSQL](https://en.wikipedia.org/wiki/NoSQL) database so you can focus on your application logic.

Traditional [relational database](https://en.wikipedia.org/wiki/Relational_database) systems like [MySQL](https://www.mysql.com/) implement indexing & scaling strategies at a platform level based on schemas defined at design time.

This guide reflects the current TypeScript‑first, inference‑driven flow: author a values‑first config literal (`as const`), optionally provide Zod schemas for your entities, and create an EntityManager via `createEntityManager(config, logger?)`. Queries are composed with a small QueryBuilder over indexes and executed across shards with automatic paging.

Please feel free to [reach out](https://github.com/karmaniverous/entity-manager/discussions) with any questions or feedback!

## What is Entity Manager?

**Entity Manager** encapsulates a **provider-agnostic, highly opinionated approach** to the [single-table design pattern](https://aws.amazon.com/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/).

> NoSQL shifts the burden of complexity from the database platform to the developer. **Entity Manager** sweeps it under the rug.

With **Entity Manager**, you can:

- **Define related data entities & structured keys** wth a simple, declarative configuration format.

- **Specify a partition sharding strategy** that maximizes query performance while permitting planned, staged scaling over time.

- **Add or remove structured index keys** from entity data objects with a single method call.

- **Perform paged, cross-shard, multi-index queries** with a single method call.

## About This Guide

Entity Manager relies heavily on TypeScript's type system to enforce the structure of your configuration. You _can_ use Entity Manager with Javascript, but Typescript will offer a much better developer experience.

Consequently, this documentation takes a _Typescript-first_ approach! All discussions & code examples will assume you are using Typescript, and we will call out Javascript-specific considerations where appropriate.

The following sections present a breakdown of this guide.

### Introduction

- Quick start (values‑first + schema‑first)

```ts
import { z } from 'zod';
import { createEntityManager } from '@karmaniverous/entity-manager';
import { defaultTranscodes } from '@karmaniverous/entity-tools';

// 1) Schema-first domain shapes (base fields only; no generated/global keys)
const emailSchema = z.object({
  created: z.number(),
  email: z.string(), // uniqueProperty
  userId: z.string(),
});

const userSchema = z.object({
  beneficiaryId: z.string(),
  created: z.number(), // timestampProperty
  firstName: z.string(),
  firstNameCanonical: z.string(),
  lastName: z.string(),
  lastNameCanonical: z.string(),
  phone: z.string().optional(),
  updated: z.number(),
  userId: z.string(), // uniqueProperty
});

// 2) Values-first config literal (as const)
const now = Date.now();
const config = {
  hashKey: 'hashKey',
  rangeKey: 'rangeKey',
  entitiesSchema: { email: emailSchema, user: userSchema } as const,
  entities: {
    email: {
      uniqueProperty: 'email',
      timestampProperty: 'created',
      shardBumps: [{ timestamp: now, charBits: 2, chars: 1 }],
    },
    user: {
      uniqueProperty: 'userId',
      timestampProperty: 'created',
      shardBumps: [{ timestamp: now, charBits: 2, chars: 1 }],
    },
  },
  generatedProperties: {
    sharded: {
      userHashKey: ['userId'],
      beneficiaryHashKey: ['beneficiaryId'],
    } as const,
    unsharded: {
      firstNameRangeKey: ['firstNameCanonical', 'lastNameCanonical', 'created'],
      lastNameRangeKey: ['lastNameCanonical', 'firstNameCanonical', 'created'],
    } as const,
  },
  indexes: {
    created: { hashKey: 'hashKey', rangeKey: 'created' },
    // ...add the rest of your index tokens as needed
  } as const,
  propertyTranscodes: {
    email: 'string',
    userId: 'string',
    created: 'timestamp',
    // ...add the rest of your transcodes as needed
  },
  transcodes: defaultTranscodes,
} as const;

export const entityManager = createEntityManager(config);
```

- [**SQL vs NoSQL**](/projects/entity-manager/sql-vs-nosql/) – A comparison of the two database paradigms and how **Entity Manager** bridges the gap.

- [**Evolving a NoSQL Database Schema**](/projects/entity-manager/evolving-a-nosql-db-schema/) – A deep dive into what a NoSQL database schema actually looks like and how **Entity Manager** simplifies the problem.

### Usage

- [**Entity Manager: A Demonstration**](/projects/entity-manager/demo/) – Presents a step-by-step Typescript implementation of a realistic data model against DynamoDB, with the help of Entity Manager.

- [**Entity Manager Configuration**](/projects/entity-manager/configuration/) – Learn how to configure **Entity Manager** to reflect your data model & scaling strategy.

## Packages

The packages in the **Entity Manager** ecosystem work together to provide a comprehensive solution for NoSQL data management.

### `entity-manager`

Provides core functionality & types for defining entities, manipulating items, and querying data.

<div class="button-row--left">
    <a href="https://github.com/karmaniverous/entity-manager" class="btn btn--primary btn--large" title="GitHub Repo"><i class="fa-brands fa-github fa-xl"></i></a>

    <a href="https://www.npmjs.com/package/@karmaniverous/entity-manager" class="btn btn--primary btn--large" title="NPM Package"><i class="fa-regular fa-heart fa-xl"></i></a>

    <a href="https://docs.karmanivero.us/entity-manager" class="btn btn--primary btn--large" title="API Docs"><i class="fa-solid fa-book fa-xl"></i></a>

    <a href="https://github.com/karmaniverous/entity-manager/discussions" class="btn btn--primary btn--large" title="Get Help!"><i class="fa-regular fa-comment fa-xl"></i></a>

</div>

### `entity-manager-demo`

A working demonstration of **Entity Manager** in action. See [the companion article](/projects/entity-manager/demo/) for a detailed walkthrough!

<div class="button-row--left">
    <a href="https://github.com/karmaniverous/entity-manager-demo" class="btn btn--primary btn--large" title="GitHub Repo"><i class="fa-brands fa-github fa-xl"></i></a>

    <a href="https://docs.karmanivero.us/entity-manager-demo" class="btn btn--primary btn--large" title="API Docs"><i class="fa-solid fa-book fa-xl"></i></a>

    <a href="https://github.com/karmaniverous/entity-manager-demo/discussions" class="btn btn--primary btn--large" title="Get Help!"><i class="fa-regular fa-comment fa-xl"></i></a>

</div>

### `entity-client-dynamodb`

Convenience wrapper for DynamoDB SDK with enhanced batch processing & **Entity Manager** support.

<div class="button-row--left">
    <a href="https://github.com/karmaniverous/entity-client-dynamodb" class="btn btn--primary btn--large" title="GitHub Repo"><i class="fa-brands fa-github fa-xl"></i></a>

    <a href="https://www.npmjs.com/package/@karmaniverous/entity-client-dynamodb" class="btn btn--primary btn--large" title="NPM Package"><i class="fa-regular fa-heart fa-xl"></i></a>

    <a href="https://docs.karmanivero.us/entity-client-dynamodb" class="btn btn--primary btn--large" title="API Docs"><i class="fa-solid fa-book fa-xl"></i></a>

    <a href="https://github.com/karmaniverous/entity-client-dynamodb/discussions" class="btn btn--primary btn--large" title="Get Help!"><i class="fa-regular fa-comment fa-xl"></i></a>

</div>

### `entity-tools`

Types & low-level functions for entity operations.

<div class="button-row--left">
    <a href="https://github.com/karmaniverous/entity-tools" class="btn btn--primary btn--large" title="GitHub Repo"><i class="fa-brands fa-github fa-xl"></i></a>

    <a href="https://www.npmjs.com/package/@karmaniverous/entity-tools" class="btn btn--primary btn--large" title="NPM Package"><i class="fa-regular fa-heart fa-xl"></i></a>

    <a href="https://docs.karmanivero.us/entity-tools" class="btn btn--primary btn--large" title="API Docs"><i class="fa-solid fa-book fa-xl"></i></a>

    <a href="https://github.com/karmaniverous/entity-tools/discussions" class="btn btn--primary btn--large" title="Get Help!"><i class="fa-regular fa-comment fa-xl"></i></a>

</div>

### `mock-db`

Mock DynamoDB-style query & scan behavior with local JSON data. Part of the **Entity Manager** test suite.

<div class="button-row--left">
    <a href="https://github.com/karmaniverous/mock-db" class="btn btn--primary btn--large" title="GitHub Repo"><i class="fa-brands fa-github fa-xl"></i></a>

    <a href="https://www.npmjs.com/package/@karmaniverous/mock-db" class="btn btn--primary btn--large" title="NPM Package"><i class="fa-regular fa-heart fa-xl"></i></a>

    <a href="https://docs.karmanivero.us/mock-db" class="btn btn--primary btn--large" title="API Docs"><i class="fa-solid fa-book fa-xl"></i></a>

    <a href="https://github.com/karmaniverous/mock-db/discussions" class="btn btn--primary btn--large" title="Get Help!"><i class="fa-regular fa-comment fa-xl"></i></a>

</div>
