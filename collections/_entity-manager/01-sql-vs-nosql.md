---
title: SQL vs NoSQL
excerpt: A well-tuned NoSQL database can outperform an RDBMS by orders of magnitude at scale... but not for free! **Entity Manager** helps close the gap.
header:
  og_image: /assets/collections/entity-manager/sql-vs-nosql-banner.jpg
  overlay_image: /assets/collections/entity-manager/sql-vs-nosql-banner-half.jpg
  teaser: /assets/collections/entity-manager/sql-vs-nosql-square.jpg
permalink: /projects/entity-manager/sql-vs-nosql/
related: true
tags:
  - aws
  - dynamodb
  - entity-manager
  - nosql
  - projects
---

<figure class="align-left drop-image">
    <img src="/assets/collections/entity-manager/sql-vs-nosql-square.jpg">
</figure>

Traditional [Relational Database Management Systems](https://en.wikipedia.org/wiki/Relational_database) like [MySQL](https://www.mysql.com/) and [SQL Server](https://www.microsoft.com/en-us/sql-server) provide a high degree of flexibility in specifying data schemas and querying data. These advantages come at the cost of performance at scale.

[NoSQL](https://en.wikipedia.org/wiki/NoSQL) databases like [DynamoDB](https://aws.amazon.com/dynamodb/) and [MongoDB](https://www.mongodb.com/) lack the helpful abstractions of RDBMS and require close attention to low-level details like indexing and partitioning to achieve high performance... but **a well-tuned NoSQL database can outperform an RDBMS by orders of magnitude at scale**.

> **Entity Manager** lets a developer achieve the scalability & performance of a NoSQL database with much of the flexibility & ease of use of a traditional RDBMS.

**Entity Manager** is a type-safe [NPM package](https://npmjs.com/karmaniverous/entity-manager) that supports opinionated schema implementation and high-performance, low-code query on NoSQL database platforms like DynamoDB.

## Schema Abstraction

A [database schema](https://en.wikipedia.org/wiki/Database_schema) specifies what kinds of data live in a database, how different data entities are related to one another, and how the data should be indexed for efficient update & query.

Database schemas can be specified at different levels of abstraction:

- **High-level schemas** employ symbolic tools like [Entity-Relationship Models](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model) and are largely platform-independent.

- **Low-level schemas** specify the exact data types and constraints for each table property or index element and are highly platform-specific.

An RDBMS allows a developer to specify a database schema largely at a high level. The database engine parses the developer's schema and implements it at a low level automatically. RDBMS engines can easily accommodate schema changes, even with data in production.

NoSQL databases offer no such intermediate layer. A developer must:

- anticipate likely data query patterns,

- create a platform-specific indexing & partitioning strategy to support those patterns, and

- support this indexing strategy by augmenting application data with specially structured data elements.

Changes to schema or index requirements require a structural refactor of this approach, plus migration & transformation of existing data to match the new schema. This is an **expensive and risky** operation on production data.

> **Entity Manager** combines a simple configuration with an opinionated process to realize the developer's high-level schema requirements as low-level data elements.

In practice this is a values‑first, TypeScript‑first flow:

- Author a single config literal (as const) that captures entities, generated properties, indexes, and property transcodes. Optionally provide Zod schemas via `entitiesSchema` to infer domain shapes.
- Create a manager with `createEntityManager(config, logger?)`. From there, helpers add/remove generated keys and typed queries orchestrate cross‑shard reads.

Since the generation of structured data elements is deterministic and reversible, Entity Manager supports **safe migration of data across schema changes.**

## Query Abstraction

Abstract query languages like [SQL](https://en.wikipedia.org/wiki/SQL) allow an RBDMS developer to interact with the database using high-level constructs. The database engine translates such high-level queries into low-level operations that it can optimize over time by generating intermediate indexes that cache low-level query results.

Whether an RDBMS developer is querying a hundred records on a single table, or a hundred million records across a dozen table partitions, the developer's query is the same. The query engine handles the complexity under the hood and stitches the result together for the developer's convenience.

**NoSQL databases offer no such abstractions.**

NoSQL queries must be adapted to the exact structures being queried. Since queries are constrained to a single data partition and index, significant queries must often be invoked in parallel across multiple partitions and indexes.

As an application scales, the developer must design an efficient partitioning & sharding strategy that:

- supports all anticipated query patterns, and

- balances performance with the hard constraints imposed by the database engine, and

- can be queried efficiently by the application.

**Entity Manager** imposes **an opinionated data partitioning & sharding strategy** on _every_ data entity, leaving the developer to specify only the scaling _schedule_ for each entity.

When it's time to query the data, **Entity Manager** reduces the problem of writing complex partitioned data queries to the composition of a simple query on one index over a single partition.

Based on the developer's schema configuration and query parameters, **Entity Manager** will:

- rehydrate any previously returned page key to extract the correct starting point for each element of the query, and

- spawn throttled, parallel queries against each relevant partition & index, and

- return a combined, deduped, sorted result set along with an efficiently dehydrated page key for the next query.

> **Entity Manager** completely decouples the way NoSQL data is queried from the way it is scaled.

## Closing the Gap

RDBMS systems have been around for a long time. These platforms are mature, and there is a vast ecosystem of tools and libraries that support them. They offer natural abstractions that make them easy to learn, efficient to implement, and safe to maintain in production.

NoSQL databases are relatively new. They offer high performance at scale, but they require a developer to understand and exploit low-level details like indexing and partitioning to achieve that performance. They offer no natural abstractions to help a developer manage the complexity of these details.

A NoSQL database is often _not_ the right choice. But when it is, these two outcomes happen more often than they should:

- **Developers will choose an RDBMS anyway**, trading a perception of safety against performance at scale. The end result is often a risky and expensive rebuild in the future as the application scales.

- **Developers will choose NoSQL but implement it wrong**, by avoiding strong patterns like the [single-table design pattern](https://aws.amazon.com/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/) out of fear of the resulting complexity. The end result is often the worst of both worlds: complexity _and_ poor performance at scale.

**Entity Manager** helps close this gap by encapsulating much of the complexity of a NoSQL implementation. This allows a developer to achieve the scalability & performance of a NoSQL database with a level of flexibility and safety approaching that of a traditional RDBMS.
