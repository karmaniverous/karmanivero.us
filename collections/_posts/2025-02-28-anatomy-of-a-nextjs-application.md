---
# prettier-ignore
title: "Anatomy of a Next.js Application"
excerpt: "Here's a useful way to think about a Next.js app that will be built by a team."
header:
  og_image: /assets/images/nextjs-anatomy-banner.jpg
  teaser: /assets/images/nextjs-anatomy-square.jpg
tags:
  - nextjs
  - principles
  - typescript
---

<figure class="align-left drop-image">
    <img src="/assets/images/nextjs-anatomy-square.jpg">
</figure>

I'm working with a new consulting client who is in a tearing rush to build a complex [Next.js](https://nextjs.org/) application in Typescript that will involve some legacy APIs and a couple of teams.

I wrote the following piece to help them think about the project in context, and it turned out sufficiently generic that I thought it might be useful to others.

So here it is: a useful way to think about building a Next.js application in a team context.

## Principles

Every implementation is an expression of a set of core engineering principles. Sometimes these are declarative; more often they are accidental. Guess which works better.

I propose that the following principles be nailed to the project front door:

- **Type Safety**. This is a Typescript project. Code should be composed in a type safe manner from the start, and project tooling should be set up to reflect that. The resulting developer experience delivers type info directly to the insertion point and eliminates a major source of churn. Automated tooling in particular should be written to deliver type safety. NO ANY TYPES, ANYWHERE!

- **Generic Architecture**. All code should be [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). WET code should immediately be abstracted into generic, type-safe components. The extra work up front end promotes high velocity down the line, and more importantly as a practice promotes better architecture.

- **Generation ≫ Authorship**. Wherever possible, type-safe code should be generated from source documents (like [OpenAPI](https://swagger.io/specification/) specs) rather than written by hand.

- **Automation ≫ Everything Else**. This applies to all aspects of the project but specifically to testing. Any test that CAN be performed by hand SHOULD be automated so it doesn’t have to be performed by hand again. If this is hard, iterate on tooling until it is not.

## Anatomy of a Next.js Application

A modern Next.js application can be divided into the following architectural layers:

- The [**Remote Back End**](#remote-back-end) is the set of APIs & services that are external to the application.

- The [**Local Back End**](#local-back-end) is the Next.js application API.

- The [**State Layer**](#state-layer) mediates access to either back end and provides client-side caching.

- The [**Component Layer**](#component-layer) renders React components & data on screen. The Component Layer is predominantly _functional_.

- The [**Design Layer**](#design-layer) imposes visual styling on the Component Layer. The Design Layer is predominantly _visual_.

### Remote Back End

The Remote Back End is generally a given, and is often only partially under the development team’s control, if at all.

The Remote Back End presents a couple of key issues:

- Remote services are generally private, and different remote services may present different authentication methods, which will usually not map cleanly to the Next.js application’s authentication method.

- Remote authentication often requires secrets (e.g. API keys) that should not be exposed in the Next.js application front end.

There are two solutions to these issues:

- Add a consolidation layer to the Remote Back End that presents an appropriate authentication interface to the Next.js application.

- Add a consolidation layer in the Local Back End that does the same thing.

Either way, the Next.js application must access Remote Back End services across APIs that may change during development. There are three ways to access these services from Typescript:

- **Directly via low-level REST libraries.** _This is a BAD solution!_ Each interface and its associated types will need to be hand-coded and maintained. This is laborious and WET, and as the load increases type safety will inevitably suffer.

- **Via published Typescript SDKs.** This nice when you can get it, but in most cases will simply not be an option.

- **Via generated Typescript SDKs.** Type-safe SDKs and associated [Zod](https://zod.dev/) schemas can be generated efficiently as part of the build process directly from OpenAPI schemas using tools like [typed-openapi](https://github.com/astahmer/typed-openapi) & [openapi-generator-cli](https://github.com/OpenAPITools/openapi-generator-cli). _This is the preferred method!_

Generating Typescript SDKs absolutely depends on the existence of related service OpenAPI specs. Therefore, **acquiring or producing remote service OpenAPI specs should be a top project priority!**

### Local Back End

In the absence of any external consolidation of the [Remote Back End](#remote-back-end), the Next.js Local Back End can perform that function.

It will almost always make sense to use the Next.js back end to support authentication, since this is the natural way to expose authentication info to server side rendering and the Local Back End. Even if use of the Local Back End is expected to be minimal NOW, it is a good idea to preserve this optionality, especially since Next.js is designed for this express purpose. [NextAuth](https://next-auth.js.org/) is the obvious tool for accomplishing this.

The Local Back End is also the appropriate place to perform any application activity that requires access to secrets that should not be exposed on the front end, or to execute workloads that are too heavy for a phone browser.

To the extent that Local Back End services will be consumed by the front end (see [State Layer](#state-layer) below), the Local Back End should generate an appropriate OpenAPI spec to be consumed by State Layer code generators.

### State Layer

In [Remote Back End](#remote-back-end) above we discussed Typescript SDK generation. The front end also needs to access services in both the local and remote back ends, with these additional constraints:

- Access will be performed from within React components, therefore it makes sense to package these components as type-safe React hooks.

- Performance on the front end requires local caching where appropriate.

React hooks & local caching for remote services can be provided by [Tanstack Query](https://tanstack.com/query/latest) (formerly React Query). Given that appropriate OpenAPI specifications exist, type-safe Tanstack hooks can be generated during development using [Orval](https://orval.dev/).

The upshot of all this is that front-end developers in the [Component Layer](#component-layer) can have direct, consistent, type-safe, cached, performant access to local & remote back-end services, directly inside their React components, without having to think very hard about where those services come from.

This promotes DRY, efficient Component-Layer code and high front-end developer velocity.

In dev environments or in the absence of fully functional back end services, the [State Layer](#state-layer) provides appropriate mocking to support front end development & testing.

### Component Layer

The Component Layer comprises the entire structure of the Next.js application front end: pages, on-page components, and supporting abstractions & libraries.

The purpose of the Component Layer is to render appropriate data and objects on the page in a box model that is...

- completely functional, and

- subject to finished styling with the addition of CSS classes and the articulation of CSS styles.

Type-safety and generic architecture a HUGE factors in the creation of a high-quality Component Layer. A little extra work up front here can deliver a FR smaller, faster, more maintainable and manageable application.

This is also the EXACT place where many teams sacrifice code quality for fast-but-sloppy execution that generates a pile of tech debt. It is NEVER a good trade.

The Component Layer should be articulated within the context of a themeable component library that provides the necessary design framework and development tooling. I like [DaisyUI](https://daisyui.com/) and [Tailwind CSS](https://tailwindcss.com/).

### Design Layer

If the [Component Layer](#component-layer) is well-executed, then the Design Layer consists almost exclusively of the set of CSS files that constitute the application theme. Developers working in this layer can iterate on the application’s visual presentation largely in isolation from developers writing functional code in the Component Layer.

## The Rest

... is confidential. But here's the point:

- Next.js (or any other platform) obviously has an _objective_ structure that is not subject to my opinions.

- Subject to that constraint, it makes sense to impose additional _conceptual_ structure that reflects the key engineering principles underlying the project and the context of available tooling and the development team.

The same method of analysis, applied to YOUR project, might yield a different result. But I think the method is pretty sound.
