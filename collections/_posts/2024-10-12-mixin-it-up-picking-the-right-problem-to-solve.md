---
# prettier-ignore
title: "Mixin It Up: Picking The Right Problem to Solve"
excerpt: "Sometimes, when you’re tangled up in a thorny problem, solving it is exactly the wrong answer."
header:
  og_image: /assets/images/mixin-it-up-banner.jpg
  teaser: /assets/images/mixin-it-up-square.jpg
redirect_from:
  - /blog/mixin-it-up-picking-the-right-problem-to-solve/
tags:
  - design
  - documentation
  - entity-manager
  - logging
  - principles
  - projects
  - typescript
---

<figure class="align-left drop-image">
    <img src="/assets/images/mixin-it-up-square.jpg">
</figure>

A couple of days ago I posted about my new [Loggable](https://github.com/karmaniverous/loggable) mixin, which makes it easy to inject an external logger into your Typescript classes at runtime.

I had a date with a pretty girl that night and was in a bit of a hurry to get the thing posted, so it wasn't until the next day when I was preparing to release the related [Batchable](https://github.com/karmaniverous/batchable) mixin that I realized I had a problem.

One of the things I love about Typescript is that it is very easy to close the gap between _good_ code and _self-documenting_ code.

**Sounds deep, right?** So what does that actually mean?

> **TL/DR:** Facing a really thorny problem? Sometimes it just means you've thrown yourself into a briar patch.

<div class="notice--info">
  <p><strong>This article turned out to be the second of a three-part series!</strong> Here they are in sequence:</p>

  <ol>
    <li><a href="/blog/loggable-a-typescript-mixin-for-generic-class-logging">Loggable: A TypeScript Mixin for Generic Class Logging</a></li>

    <li><strong>Mixin It Up: Picking the Right Problem to Solve</strong> (you are here)</li>

    <li><a href="/blog/composition-in-action-finishing-the-swing">Composition in Action: Finishing the Swing</a></li>

  </ol>
</div>

## What is Good Code?

Well, clearly, at a minimum good code fulfills requirements: it does what it was meant to do.

But every grown-up developer knows there is more to it than that. **_Really_ good code is more than just a dropped plate of spaghetti that happens to stick to the wall.** It also meets a coherent set of engineering principles, like:

- [**SOLID**](https://en.wikipedia.org/wiki/SOLID) - Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion
- [**DRY**](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) - Don't Repeat Yourself
- [**KISS**](https://en.wikipedia.org/wiki/KISS_principle) - Keep It Simple, Stupid
- [**YAGNI**](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it) - You Ain't Gonna Need It (so don't write it)
- [**Law of Demeter**](https://en.wikipedia.org/wiki/Law_of_Demeter) - Only talk to your immediate friends
- [**Composition over Inheritance**](https://en.wikipedia.org/wiki/Composition_over_inheritance) - Prefer composition to inheritance

Good code encapsulates the understanding that **the work is not done**: _my_ good code will eventually have to talk to _your_ good code, and it behooves us both to make that conversation as easy as possible.

## What is Self-Documenting Code?

The trivially naive view of self-documenting code is that it is code so clear and well-organized that it doesn't need comments.

> Pro tip: adhering strictly to this view is a great way to get yourself fired. **Especially by me.**

In the real world we all use advanced IDEs like Visual Studio Code, which do neat things like show you a snippet of useful documentation when you're about to invoke a function. Partly that happens automagically thanks to type inference (unless you're using Javascript, in which case 🤷‍♂️), but many of the more useful bits (like what is this parameter actually _for_) only show up because you put them there on purpose.

In Typescript, this means you took the trouble to document your code with structured [TSDoc](https://tsdoc.org/) comments. In Javascript, you took a lot _more_ trouble to do the same thing with [JSDoc](https://jsdoc.app/). Either way, your code doesn't just speak to other developers. It speaks to your _tools_.

Here's an example, an interface declaration from [`Loggable`](https://github.com/karmaniverous/loggable) tarted up with TSDoc:

```ts
/**
 * Loggable options.
 */
export interface LoggableOptions {
  /** Identifies logger endpoints disabled when `enableAll !== true`. */
  disabled: string[];

  /** Enables all logger endpoints when `true`. */
  enableAll: boolean;
}
```

Here's what I see when I hover over one of those properties elsewhere in my code:

{% include figure image_path="/assets/images/mixin-it-up-intellisense.png" caption="_TSDoc-driven intellisense._" popup=true %}

So far, par for the course.

But the beauty of living here in the future is that, you've taken the trouble to make your documentation accessible to your tools, your options are only limited by your tools... and your toolbox is expanding by the day!

For example, all of my Typescript projects use [TypeDoc](https://typedoc.org/), which can read my TSDoc comments and automagically generate API documentation for my project. Here's [the page](https://docs.karmanivero.us/loggable/interfaces/loggable.LoggableOptions.html) generated by the interface declaration above:

{% include figure image_path="/assets/images/mixin-it-up-tsdoc.png" caption="_TypeDoc API documentation._" popup=true %}

So with regard to the naive view proposed above: of _course_ your code should be well-organized and clear. That doesn't mean you should skip the comments. It just means that your comments should be attached to something worth documenting.

_Self-documenting_ code, on the other hand, is another animal entirely.

> Self-documenting code is code that can participate actively in its own documentation!

Now _that_ is deep.

## A Brief Detour into Mixins

If you recall, we started this conversation because I ran into some documentation-related problem with my Batchable mixin implementation. But wait... what exactly is a _mixin_?

Before we dig into the actual problem, let's take a moment to understand the distinction between a **mixin** and a **class**.

A [Typescript class](https://www.typescriptlang.org/docs/handbook/2/classes.html) is a way to encapsulate a complex set of behaviors and data into a single, reusable unit. The Typescript version adheres pretty faithfully to the Object-Oriented Programming (OOP) paradigm, with the caveat that "class" inheritance in Typescript is really [prototype-based](https://en.wikipedia.org/wiki/Object-oriented_programming#Prototype-based).

Compared to a more "pure" OOP language like Ruby or (shaddap haters) C++, a key constraint imposed by Typescript's prototype-inheritance model is _single inheritance_. This means that a class can only inherit from one other class, as opposed to the _multiple inheritance_ allowed in those other languages.

For example, let's say I have a `Loggable` class that knows how to log to an external logging system, and a `Batchable` class that knows how to batch & throttle data operations against an API. In a pure OOP language like Ruby, I could create an `EntityManager` class that inherits from both `Loggable` and `Batchable`, and get the combined functionality of both.

{% include figure image_path="/assets/diagrams/mixin-it-up-multiple-inheritance.png" caption="_Multiple inheritance in pure OOP languages._" %}

In Typescript, I can't do that. I can only inherit from a _single_ class, so I have to find some way to...

- Preserve the useful isolation of distinct packages of functionality into the `Loggable` and `Batchable` classes.

- Combine them into a _single_ class that `EntityManager` can inherit.

This is where [**mixins**](https://medium.com/@saif.adnan/typescript-mixin-ee962be3224d) come in.

A _mixin_ is a **function** that takes a **class** as an argument and returns a **new class** that combines the functionality of the original class with the functionality of the mixin. In the degenerate case, a mixin takes an empty class as its argument and returns a new class that exhibits _only_ the mixin functionality.

Here's what the same scenario looks like using mixins instead of multiple inheritance:

{% include figure image_path="/assets/diagrams/mixin-it-up-single-inheritance.png" caption="_Single inheritance with mixins in Typescript._" popup=true %}

The salient point here is that when I made the decision to implement `Loggable` and `Batchable` as mixins, the things that I published were no longer _classes_, but _functions_.

## The Problem

We've already established that tools like TypeDoc can activate the capacity of self-documenting code to do really useful things like automagically generate complex API documentation.

Could I do this by hand? Sure... but it's such a pain that I probably wouldn't bother. And even if I did, I'd only do it once. As soon as I made a change to the code, the documentation would be out of sync, and the cost of keeping the documentation in sync with the code would be so high that I wouldn't do it for long.

This prompts an observation so obvious that it's easy to overlook: the most valuable tools come with a built-in set of constraints. In other words, **the more I like what TypeDoc does with my code, more important it is to stick to code TypeDoc can actually digest.**

So what's the problem?

To make any use of a Typescript class, I need to know a great deal about its internal structure. Therefore, TypeDoc takes great pains to document the properties, methods, and types related to any class.

For example, here's the top of the `EntityManager` class page generated by TypeDoc:

{% include figure image_path="/assets/images/mixin-it-up-entity-manager-class.png" caption="_TypeDoc API documentation for the `EntityManager` class._" popup=true %}

But a mixin is not a class. It's a _function_ that returns a class. And TypeDoc treats functions very differently: you get the function signature and any extra documentation you might have added at the top of the function, but that's it.

Period.

Here's the top of the current `Loggable` mixin function page generated by TypeDoc:

{% include figure image_path="/assets/images/mixin-it-up-loggable-mixin.png" caption="_TypeDoc API documentation for the `Loggable` mixin function._" popup=true %}

**Half of what the user actually cares about is completely missing:** the `logger` and `loggableOptions` properties generated on the `Base` class by the mixin. Without this information, the documentation is incomplete and the mixin is effectively useless to anybody but yours truly.

So, mixins... sweet implementation pattern, for sure. But given a mixin's dual nature as both a function and a class, how can we make sure that the documentation generated by TypeDoc is complete?

## The Problem Behind the Problem

So far, I've presented this problem from a very particular perspective: _I have good reasons to do an advanced thing and am running up against the constraints of my toolbox_.

But [way up at the top of this article](#what-is-good-code), we articulated a list of engineering principles that good code should adhere to. And one of those principles was [**Composition over Inheritance**](https://en.wikipedia.org/wiki/Composition_over_inheritance).

Ring a bell?

Recall that the entire topic of mixins is only relevant because I wanted to overcome Typescript's multiple-inheritance constraint and inherit functionality in a single class from both `Loggable` and `Batchable`. Mixins are definitely one answer... but, as the discussion above shows, they bring their own set of problems along for the ride.

**So what if we shouldn't even be having this discussion?**

What if core engineering principles _really matter?_ In that case, the compounding difficulties imposed by the mixin solution aren't just a deeper set of problems to be solved. They're a [_design smell_](https://en.wikipedia.org/wiki/Design_smell), and stand as **clear evidence that I'm barking up the wrong tree!**

Mixins solve an _inheritance_ problem. If I can figure out how to convert my inheritance problem into a _composition_ problem, I won't have _solved_ my mixin documentation problem...

I'll have _eliminated_ it.

## The Composition Solution

If extreme difficulty can be a sign that you're on the wrong track, then what about the reverse?

Watch this:

- The `Loggable` mixin generates a class with the `loggableOptions` and `logger` properties. _So rip the class out of its mixin function and publish it as a standalone class._

- The `Batchable` mixin generates a class with the `batchableOptions` property and the `processBatch` method. _So rip the class out of the mixin function and publish it as a standalone class._

- _Pass an instance of each class as a constructor argument, store it as an instance property, and reference it on any class where it's needed._

- TypeDoc will document the properties and methods of the `Loggable` and `Batchable` classes as it would any other class. _Nothing to see here, move along._

That's it. **That's the whole solution.**

Here's what the combined implementation looks like now:

{% include figure image_path="/assets/diagrams/mixin-it-up-composition.png" caption="_Composition over inheritance!_" %}

... at least it will once I've finished pushing things around this week.

## Cue The Music

So what's the takeaway here?

> **Sometimes, when you're tangled up in a thorny problem, solving it is exactly the wrong answer.**

Instead, take a step back and ask yourself if you're even solving the right problem.

By revisiting core engineering principles like _Composition over Inheritance_, we can eliminate unnecessary complexity and arrive at cleaner, more maintainable solutions.

In other words, **instead of hacking your way through the briar patch, spread your wings and fly!**
