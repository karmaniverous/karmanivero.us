---
# prettier-ignore
title: "Composition in Action: Finishing the Swing"
excerpt: "In software engineering, what look to be the easy parts often turn out to be the hard parts. If you're lucky, the reverse is also true, but don't count on it."
header:
  og_image: https://github.com/karmaniverous/controlled-proxy/raw/main/assets/controlled-proxy-banner.jpg
  teaser: https://github.com/karmaniverous/controlled-proxy/raw/main/assets/controlled-proxy-square.jpg
redirect_from:
  - /blog/composition-in-action-finishing-the-swing/
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
    <img src="https://github.com/karmaniverous/controlled-proxy/raw/main/assets/controlled-proxy-square.jpg">
</figure>

In software engineering, what look to be the easy parts often turn out to be the hard parts. If you're lucky, the reverse is also true, but don't count on it. 🤣

This has certainly been the case with the Typescript refactor of [`entity-manager`](https://github.com/karmaniverous/entity-manager), which I proclaimed [largely code complete](https://karmanivero.us/blog/code-complete-a-day-in-the-life-of-a-product/), oh, about a month ago. The job since then was supposed to be largely about documentation, testing, and encapsulating key [cross-cutting concerns](https://en.wikipedia.org/wiki/Cross-cutting_concern) like injectable logging.

Well, if you've been following along, **you know that plan blew up in my face**.

<div class="notice--info">
  <p><strong>This article turned out to be the last of a three-part series!</strong> Here they are in sequence:</p>

  <ol>
    <li><a href="/blog/loggable-a-typescript-mixin-for-generic-class-logging">Loggable: A TypeScript Mixin for Generic Class Logging</a></li>

    <li><a href="/blog/mixin-it-up-picking-the-right-problem-to-solve">Mixin It Up: Picking the Right Problem to Solve</a></li>

    <li><strong>Composition in Action: Finishing the Swing</strong> (you are here)</li>

  </ol>
</div>

First I thought I'd encapsulate logging, batching, and related services in some reusable base classes.

This led me head-first into Typescript's single-inheritance constraint, whereupon I decided to [implement these services as mixins](/blog/loggable-a-typescript-mixin-for-generic-class-logging/).

That produced the[ `loggable`](https://github.com/karmaniverous/loggable) library, and seemed to work nicely... until I looked at my generated documentation and realized that [TypeDoc doesn't know what to to with a mixin](/blog/mixin-it-up-picking-the-right-problem-to-solve/). Plus the patterns I'd need to _consume_ those mixins were exceedingly ugly. Not a good sign.

But if you click that last link, you'll also see that I read this result as a strong argument favoring [_Composition over Inheritance_](https://en.wikipedia.org/wiki/Composition_over_inheritance), which is one of those core engineering principles I so often wish I'd been paying attention to all along.

> **Epiphany #1:** We thought we were solving an _inheritance_ problem, but we're really solving a _composition_ problem!

Now I'm an old guy, which means that pattern-recognition is more of a thing for me than for a younger engineer. It ain't magic: I've just probably seen a lot more patterns than you have.

So here's a rule of thumb that works pretty well for me: **Things get easier and more generic when I'm on the right path!**

Case in point: once I decided that composition was the _only_ way I was going to approach the `loggable` problem, the solution fell right into my lap.

And it's _awesome_.

## Logging Requirements

Let's forget about the `batchable` stuff and focus strictly on `loggable`. As a recap from previous articles, here's what we are trying to accomplish.

Say we want to write a widget that does some internal logging. Then...

- **Inside our widget, the logger should behave in a familiar way.** So we should be able to cherry-pick the behavior we want from some default logger (e.g. `console`).

- **We _don't_ want to have to specify the _actual_ logger our widget uses until runtime.** So we should be able to inject it from the outside, and as long as we inject a logger that's compatible with the bits we were using on the inside, it should work.

- **We _don't_ want to have to wait until runtime to discover we've passed our widget an inadequate logger.** So the whole thing should be [type-safe](https://clouddevs.com/typescript/type-safety/), and an incompatible logger should throw a compile-time Typescript error.

- **If our widget depends on other widgets that also log, it should be able to pass its logger on to them.** So this injectability characteristic should be commutative.

- **We want to be able to alter the behavior of the logger from _outside_ our widget, for example to turn off annoying internal debug logging but leave error logging intact, _without_ affecting external logging.** So the _logger_ and the associated _configuration_ should be distinct. I should be able to pass the same logger around with different configurations in different places.

- **Logging engines are expensive, so whichever one we are using, we never want to have more than one of them.** So I should pass my logger around _by reference_, and I should _definitely_ not be creating any clones of the thing, whether deep or shallow!

On the face of it, that seems like a tall order. But there's something interesting about that list of requirements, and I wonder if you picked up on it.

Give up? **Except for the examples I gave, _none_ of those requirements are intrinsically specific to logging!** They could just as usefully apply to _any_ injected dependency.

> **Epiphany #2:** We're not solving a _logging_ problem. We're solving a _dependency injection_ problem!

Groovy.

And once you start digging around for approaches to dependency injection via composition in Javascript, it won't be too long until you run into the [Proxy object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy).

## The Proxy Object

A Javascript Proxy object enables you to create a proxy for another object, which can intercept and redefine fundamental operations for that object.

To illustrate, say you tried to run this code:

```ts
console.foo('bar); <== oops!
```

You'd get a `TypeError` because `console` doesn't have a `foo` method. But what if you could intercept that call and do something else with it?

```ts
const consoleProxy = new Proxy(console, {
  get: function (target, prop, receiver) {
    if (prop in target) {
      return target[prop];
    } else {
      return function () {
        console.log(
          `You tried to call ${prop} on console, but it doesn't exist!`
        );
      };
    }
  },
});
```

Now if you run the same code:

```ts
consoleProxy.foo('bar);
// You tried to call foo on console, but it doesn't exist!
```

**_How cool is that??_**

You can think of a Proxy object as a kind of _middleware_ that sits between your code and the object you're trying to interact with. It can intercept and modify the behavior of that object in all sorts of ways.

And here's an important point: the Proxy object _does not care_ what kind of object it proxies! It will expose whatever features the underlying object has, with whatever modifications and extensions you care to make.

So how does this help us with our dependency injection problem?

> **Epiphany #3:** A Proxy object is a universal dependency injector!

## Solving The Problem From Both Ends

We have two key requirements that are kind of opposed to one another:

- From the _inside_, our logger should appear exactly the way we want it to, but...

- From the _outside_, it should be whatever we inject. As long as it's compatible.

So here's a strategy:

1. **Make our widget require an injected dependency (like a proxied logger) with a particular shape**, and complain if we give it something different.

2. **Create a function that proxies a dependency (like a logger)**, tries to cram it into some shape alongside a control configuration, and complains if the dependency won't fit.

3. **Introduce the proxied dependency to our widget**. if they get along, then the widget will play with its internal proxy, and the proxy will play with the external dependency.

Still groovy! And, although the application I had in mind was logging, this thing would work for _any_ injectable dependency!

All I had to do was build it.

## Introducing `controlledProxy`

`controlledProxy` allows the behavior of any object to be modified & controlled non-destructively at runtime. It's a **universal dependency injector** that can be used to solve a wide variety of problems.

{% include figure image_path="https://github.com/karmaniverous/controlled-proxy/raw/main/assets/controlled-proxy.png" caption="_`controlledProxy` in a nutshell._" popup=true %}

<div class="button-row--left">
    <a href="https://docs.karmanivero.us/controlled-proxy/" class="btn btn--info btn--large">API Docs</a>

    <a href="https://github.com/karmaniverous/controlled-proxy" class="btn btn--primary btn--large"><i class="fa-brands fa-github fa-2xl"></i></a>

    <a href="https://www.npmjs.com/package/@karmaniverous/controlled-proxy" class="btn btn--primary btn--large"><i class="fa-brands fa-npm fa-2xl"></i></a>

</div>

### Installation

```bash
npm install @karmaniverous/controlled-proxy
```

### Basic Usage

The `controlledProxy` function creates a type-safe proxy of any `object`.

The [`options`](https://docs.karmanivero.us/controlled-proxy/interfaces/controlled_proxy.ControlledProxyOptions.html) parameter is an object with the following properties:

| Property                        | Type                                                                                                                      | Default           | Description                                                                                                                                                                                                                                               |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `defaultControls`               | `Record<PropertyKey, boolean>`                                                                                            | `{}`              | A map of controlled property keys to boolean values. When this value is `true` or the property is uncontrolled, the property will behave normally. When this value is false, the property will execute the disabled member handler or return `undefined`. |
| `defaultDisabled-MemberHandler` | [`DisabledMemberHandler`](https://docs.karmanivero.us/controlled-proxy/types/controlled_proxy.DisabledMemberHandler.html) | `() => undefined` | A function that is called when a disabled controlled property is accessed.                                                                                                                                                                                |
| `target`                        | `object`                                                                                                                  | _required_        | The object to proxy.                                                                                                                                                                                                                                      |

```ts
import { controlledProxy } from "@karmaniverous/controlled-proxy";

// Create a controlled console logger. Info messages are disabled by default.
const controlledConsoleLogger = controlledProxy({
  defaultControls: { debug: true, info: false },
  target: console,
});

// Log messages.
controlledConsoleLogger.debug("debug log");
controlledConsoleLogger.info("info log");
// > debug log
```

### Runtime Control

The proxy object has two special properties, keyed with symbols that can be imported from the package:

| Property                       | Type                                                                                                                      | Description                                                                                                                                                                                                                                               |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `[controlProp]`                | `Record<PropertyKey, boolean>`                                                                                            | A map of controlled property keys to boolean values. When this value is `true` or the property is uncontrolled, the property will behave normally. When this value is false, the property will execute the disabled member handler or return `undefined`. |
| `[disabledMember-HandlerProp]` | [`DisabledMemberHandler`](https://docs.karmanivero.us/controlled-proxy/types/controlled_proxy.DisabledMemberHandler.html) | A function that is called when a disabled controlled property is accessed. Defaults to `() => undefined`.                                                                                                                                                 |

```ts
import {
  controlledProxy,
  controlProp,
  disabledMemberHandlerProp,
} from "@karmaniverous/controlled-proxy";

// Create a controlled console logger. Info messages are disabled by default.
const controlledConsoleLogger = controlledProxy({
  defaultControls: { debug: true, info: false },
  target: console,
});

// Disable debug messages & enable info messages at runtime.
controlledConsoleLogger[controlProp].debug = false;
controlledConsoleLogger[controlProp].info = true;

// Log messages.
controlledConsoleLogger.debug("debug log");
controlledConsoleLogger.info("info log");
// > info log

// Change the disabled member handler.
controlledConsoleLogger[disabledMemberHandlerProp] = (
  target: Console,
  prop: PropertyKey
) => target.log(`Accessed disabled member: ${prop.toString()}`);

// Log messages again.
controlledConsoleLogger.debug("debug log");
controlledConsoleLogger.info("info log");
// > Accessed disabled member: debug
// > info log
```

### Proxy Injection

Here's an example of the real power of the library: **let's inject a controlled proxy into a class!**

```ts
import {
  controlledProxy,
  controlProp,
} from "@karmaniverous/controlled-proxy";

// Create a class that accepts a proxied logger as a constructor argument.
class MyClass {
  // Proxied logger must be compatible with console.debug & console.info.
  constructor(private logger: Pick<Console, "debug" | "info">) {}

  // Exercise the proxied logger.
  myMethod() {
    this.logger.debug("debug log");
    this.logger.info("info log");
  }
}

// Create a controlled console logger, with all messages enabled by default
// and a custom disabled member handler.
const controlledConsoleLogger = controlledProxy({
  defaultControls: { debug: false, info: true },
  defaultDisabledMemberHandler: (
    target: Console,
    prop: PropertyKey
  ) => target.log(`Accessed disabled member: ${prop.toString()}`),
  target: console,
});

// Instantiate the class with the controlled console logger.
const myConsoleInstance = new MyClass(controlledConsoleLogger);

// Disable console debug messages at runtime.
controlledConsoleLogger[controlProp].debug = false;

// Exercise the proxied console logger from within the class.
myConsoleInstance.myMethod();
// > Accessed disabled member: debug
// > info log

// Create an equivalent controlled winston logger, with all messages enabled by
// default and a custom disabled member handler.
import { createLogger, type Logger } from "winston";

const controlledWinstonLogger = controlledProxy({
  defaultControls: { debug: true, info: true },
  defaultDisabledMemberHandler: (
    target: Logger,
    prop: PropertyKey
  ) =>
    target.log(
      "warn",
      `Accessed disabled member: ${prop.toString()}`
    ),
  target: createLogger(),
});

// Instantiate the class again with the controlled winston logger.
const myWinstonInstance = new MyClass(controlledWinstonLogger);

// Disable winston debug messages at runtime.
controlledWinstonLogger[controlProp].debug = false;

// Exercise the proxied winston logger from within the class.
myWinstonInstance.myMethod();
// > [winston] { "level":"warn", "message":"Accessed disabled member: debug" }
// > [winston] { "level":"info", "message":"info log" }
```

## Conclusion

It's fun to show off work you're proud of, and `controlledProxy` is no exception. It's a neat solution to a deceptively hard problem, and I hope it gets some traction out there.

It also solves my _own_ hard problem, which is cool... all the other dependency injectors I found out there carried far more dependency baggage than I cared to inherit, and this widget feels like it strikes a nice balance.

But that's not why we're here, is it?

I hope the three articles in this series have served as a useful illustration of how real software gets done:

- [This one](/blog/loggable-a-typescript-mixin-for-generic-class-logging) shows that even a VERY experienced engineer can get seduced away from core engineering principles by a shiny new toy,

- [This one](/blog/mixin-it-up-picking-the-right-problem-to-solve) shows how to use those same principles to think yourself back on track,

- And the one you're reading now shows what you can do with those principles once you've got them well in hand.

Do good work! 🚀
