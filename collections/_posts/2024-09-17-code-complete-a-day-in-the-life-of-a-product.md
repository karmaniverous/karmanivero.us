---
# prettier-ignore
title: "Code Complete: A Day in the Life of a Product"
excerpt: "We’re all professionals here, right? And we all know there’s often quite a gap between what gets posted to social media and the reality on the ground."
header:
  og_image: /assets/images/code-complete-tests-preview.jpg
  teaser: /assets/images/code-complete-tests.jpg
redirect_from:
  - /blog/code-complete-a-day-in-the-life-of-a-product/
tags:
  - aws
  - design
  - documentation
  - dynamodb
  - entity-manager
  - projects
  - typescript
toc: true
---

<figure class="align-left drop-image">
    <img src="/assets/images/code-complete-tests.jpg">
</figure>

Today was a _big_ day for me. After a solid month of work, I completed the Typescript refactor of [`entity-manager`](https://github.com/karmaniverous/entity-manager) and its supporting libraries! All tests are passing and the new major release is now [live on NPM](https://www.npmjs.com/package/@karmaniverous/entity-manager).

But... **we're all professionals here, right?** And we all know there's often quite a gap between what gets posted to social media and the reality on the ground.

**So what does _code complete_ actually _mean?_**

## Eating My Own Dog Food

`entity-manager` encapsulates the [single-page design pattern](https://aws.amazon.com/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/) for [NoSQL databases](https://en.wikipedia.org/wiki/NoSQL). It is:

- **Configuration-driven.** You define your data schema and indexing strategy in a config object. All the specially formatted data elements required to _implement_ your config are generated & managed for you under the hood.

- **Massively scalable.** [Partition sharding](https://medium.com/@_amanarora/partitioning-sharding-choosing-the-right-scaling-method-dbc6b2bec1d5) is part of your config. Your sharding strategy can scale with your data, and you can query a hundred million records in parallel over four indexes across 32 partition shards with the same command that queries one index on a single partition.

- **Provider-agnostic.** The underlying database query is an [injected dependency](https://en.wikipedia.org/wiki/Dependency_injection), so while I built `entity-manager` for [AWS DynamoDB](https://aws.amazon.com/pm/dynamodb), you could run it against any NoSQL database engine.

- **Type-safe.** If you use Typescript, it is nearly impossible to create an invalid configuration. If you don't, the library will still catch most config errors at runtime (thanks [`zod`](https://github.com/colinhacks/zod)!).

Now why would I do all that?

At my real job, I'm building a massively scalable product with a complex data model that is (at last count) spread across 16 distinct back-end services, each with its own DynamoDB data store. If I had built [VeteranCrowd](https://veterancrowd.com) without a dependency like `entity-manager` in place, we would now have a dangerously fragmented code base and would be facing a complete rebuild as the application scales.

So I built `entity-manager` because I needed it. And I built it as a hyper-generic, open-source library because I've learned the hard way that there's a clear difference between building a thing to solve a problem, and building a thing to solve a _category_ of problems.

When you create a generic solution to an entire category of problems, you don't just build the thing differently. **_You build a different thing._**
{: .notice--info}

My initial implementation of the VeteranCrowd back end was in Javascript, so early versions of `entity-manager` followed suit.

But early on we refactored our front end into Typescript, and the [Serverless Framework](https://www.serverless.com/) underlying our back end has shifted to Typescript-first with [version 4](https://www.serverless.com/blog/serverless-framework-v4-general-availability). So like it or not (and I _do_ like it), our back end will be moving to Typescript soon enough.

If we're going to do that right, our key back-end dependencies need to be [type-safe](https://dev.to/mistval/type-safe-typescript-4a6f). And I'm now sitting on two years' worth of lessons learned from operating the VeteranCrowd back end in production. Since `entity-manager` is about as key as key dependencies get, it seemed like the right place to start.

So I rebuilt `entity-manager` from the ground up in Typescript.

And even though VeteranCrowd's back end is still a Javascript implementation, I can import the new version of `entity-manager` _immediately_—with just a minimal refactor—in order to take advantage of its simplified configuration and significant performance enhancements.

That's a win!

## The Documentation Gap

If you wanted to explore the [`entity-manager` repo](https://github.com/karmaniverous/entity-manager) right now, there'd be good news and bad news.

The good news is that it _works_. If you want to manage & query NoSQL data at scale, `entity-manager` is a solid choice.

Also, thanks to [TypeDoc](https://typedoc.org/), the `entity-manager` API is reasonably well-documented. My [build process](https://github.com/karmaniverous/npm-package-template-ts) generates a very nice [API reference](https://docs.karmanivero.us/entity-manager/) directly from the Typescript source code, so if you don't mind doing a little digging, you could probably get yourself up and running without too much trouble.

The bad news is that there's a whole layer of _conceptual_ documentation that as yet just doesn't exist.

In order to use `entity-manager` effectively, you need to understand:

- The [single-page design pattern](https://aws.amazon.com/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/)

- Partition sharding strategies and how to [configure](https://docs.karmanivero.us/entity-manager/interfaces/entity_manager.ShardBump.html) them.

- How to define an [Entity](https://docs.karmanivero.us/entity-manager/types/entity_manager.ConfigEntity.html)

- How to define [generated properties](https://docs.karmanivero.us/entity-manager/types/entity_manager.ConfigEntityGenerated.html) & indexes

- [Transcodes](https://docs.karmanivero.us/entity-manager/types/entity_manager.Transcodes-1.html) and how to connect them to generated property & index elements

- How to write a [ShardQueryFunction](https://docs.karmanivero.us/entity-manager/types/entity_manager.ShardQueryFunction.html) & pass it to the [`EntityManager`](https://docs.karmanivero.us/entity-manager/classes/entity_manager.EntityManager.html) class

- How to [add generated keys](https://docs.karmanivero.us/entity-manager/classes/entity_manager.EntityManager.html#addKeys) to your data items & [remove them](https://docs.karmanivero.us/entity-manager/classes/entity_manager.EntityManager.html#removeKeys).

- How to [query your data](https://docs.karmanivero.us/entity-manager/classes/entity_manager.EntityManager.html#query).

If you followed any of those links, you're rolling your eyes right now. 🤣

I built `entity-manager`, so of course I know how to use it. And, truthfully, using it is pretty straightforward... _once you know how_.

**⭐ Everything's easy once you know how!**
{: .notice--info}

In order to get there, you're going to need _real_ documentation, with explanations in English, examples in code, and diagrams to help you understand the concepts. **_Lots_ of diagrams.**

Without that documentation—and, critically, without all those pictures—there isn't a chance in the world that anybody but me is ever actually going to use `entity-manager`.

So that's what comes next: I need to write the book on `entity-manager`.

Now... **_just wait a minute!_**

A _product_ is something people will pay money for, right? And so far the only actual product we've discussed here is VeteranCrowd. If VeteranCrowd is the product, then I already have everything I need, don't I?

I do very much live in the open-source world, and `entity-manager` is a very useful open-source tool. I'm proud of it... but it's _not_ a product. Spending another month writing `entity-manager` documentation is hardly going to pay my beer tab.

I don't work this hard for free. **So what's the actual product?**

## Still Not The Product

Thanks to tools like [AWS Lambda](https://aws.amazon.com/pm/lambda), DynamoDB, and the Serverless Framework, a competent developer with a running start can put a working, data-enabled API into production in about thirty-seven seconds, give or take.

That's the good news.

The bad news is that **a working, data-enabled API is not an _application_.**

Real serverless applications have _lots_ of moving parts, including:

- Multiple back-end services that can securely talk to the front end and each other.

- A rational authentication scheme that works across all services.

- A service layer that does all the things required by _every_ service, in a consistent way (hello `entity-manager`!).

- Coherent architectural & design patterns that unify all those services so they can be managed with a single set of tools.

- An infrastructure that can move all the parts from development through test and into production with a maximum of automation and a minimum of fuss.

**_... at scale!_**

A couple of years ago I published a Javascript-based [AWS API Template](/blog/aws-api-template/). This was an attempt to bring all these parts together into a template for a real serverless application, and I used it as the starting point for the VeteranCrowd back end.

Two years of build & production across 16 services have taught me a _lot_.

Some of it, naturally, is application-specific, and **clearly there is no percentage in sharing my employer's secrets**.

But most of the really big lessons I've learned in the last couple of years would apply equally well to _any_ real-world, massively scalable serverless application. And as I've learned these lessons, I've been careful to organize them into hyper-generic, open-source libraries like `entity-manager`.

**⭐ You haven't _really_ learned a lesson until you've encapsulated it in code.**
{: .notice--info}

So... **_what's the product?_**

Once I complete & document Typescript refactors of my remaining key dependencies, I will be in a position to publish a new version of the AWS API Template that...

- Is implemented entirely in Typescript.

- Leverages libraries like `entity-manager` that encapsulate highly opinionated, battle-tested approaches to all the hard things.

- Solves the infrastructure problem in a detailed, opinionated way, right out of the box.

- Is _fully documented_ with conceptual explanations, code examples, and diagrams.

Super useful, right? A robust head-start for any organization that wants to build a complex, massively-scalable serverless application in the real world, **at a savings of tens or hundreds of thousands of dollars** in development costs.

And totally open source! So... **_still not the product!_**

## The Actual Product

That refactored AWS API Template is going to be a bit of a monster. It will take some time to put together. And I _do_ work for a living!

But **my deal with my employers is always the same**: in order to build your thing as fast as possible and at the highest possible quality, as a matter of principle I build it out of highly generic parts.

The _product_ and its _configuration_ belong to my employer. The _parts_ belong to me... and, since I always open-source them, they belong to everybody.

**Everybody wins.**

All those parts will be well-documented, and the template as a whole will _also_ be well-documented, but there's going to be a lot of territory to cover. Even with the best documentation in the world, it'll be awfully helpful to have a native guide.

That's me.

Twenty years ago, I would have announced that I'm writing a book. But today, I'm not writing a book. **I'm writing a _course_.**

The course will be a series of video lectures presenting the AWS API Template in detail, from the ground up, and drawing heavily on the documentation I had to write anyway. **The goal will be to get a developer up & running with a working API and supporting infrastructure in just a few hours**, with a clear understanding of the template, the underlying concepts, and everything happening under the hood.

I'll sell the course on [Udemy](https://www.udemy.com/) or a similar platform, and the price tag will be a nice payoff for me but **an orders-of-magnitude win** for any organization that is serious about building a serverless application in the real world.

**THAT is the product.**

Expect the course to be available within the next few months. I'll be sure to post updates here as the effort proceeds.

Meanwhile, if you're building a serious serverless application _now_ and don't want to wait months to take my course, [get in touch](https://calendly.com/karmaniverous)!

The parts are all there, they're battle-tested, and I'm always happy to help.
