---
# prettier-ignore
title: "Radash: The New Lodash"
excerpt: "If you use Typescript and are still using Lodash: stop. It's time for a change Give Radash a try!"
header:
  og_image: /assets/images/radash-the-new-lodash.png
  teaser: /assets/images/radash-the-new-lodash.png
redirect_from:
  - /blog/radash-the-new-lodash/
tags:
  - typescript
toc: false
---

<figure class="align-left drop-image">
    <img src="/assets/images/radash-the-new-lodash.png">
</figure>

Back before the dawn of recorded history—say, before early 2015—things were _different_.

Typescript existed, but nobody outside Microsoft actually used it. Javascript was the undisputed king of the development world, and it was freaking UGLY. Before the mid-2015 release of ES6, Javascript didn't have:

- [Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [Default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)
- [Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)
- [Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
- [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) & [`async`/`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
- [Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- Multi-line strings
- `let` or `const`

You could still do all the things, but getting them done was _painful_. Which was way Lodash came along in 2012.

[Lodash](https://lodash.com/) is a functional library that made Javascript WAY less painful. Lodash functions run the gamut, from simple staples like type conversions, all the way to super tricky moves like generic [function currying](https://www.linkedin.com/pulse/what-currying-javascript-arjunan-k/) & [deep object cloning](https://stackoverflow.com/questions/728360/how-do-i-correctly-clone-a-javascript-object). In the main, Lodash functions just _work_, and they are _fast_... often faster than their native Javascript equivalents.

Small wonder Lodash was the single most popular package on NPM pretty much forever, with almost 200k dependent packages and a whopping 9.6 million downloads every week!

Despite all that, there are some issues.

Lodash is big. Like, _really_ big. You can use bits and pieces of it, but the library has a lot of internal dependencies and the really useful bits don't tree-shake very well. So let's just say your decision to lean on Lodash won't make your bundle any _smaller_.

It's also a victim of its own success. Many key Lodash functions were added to the Javascript standard library in ES6 and later Javascript releases, which means that Lodash is now a bit redundant. Lodash can still do a lot of things Javascript can't do without jumping through some hoops, but you just don't _need_ Lodash the way you used to.

But here's the real kicker. Back in 2015, when Javascript started catching up to Lodash with the launch of ES6, almost nobody used Typescript. So the fact that Lodash was written in Javascript was not an issue, and in the Javascript world [type safety](https://www.baeldung.com/cs/type-safety-programming#:~:text=Type%20safety%20in%20the%20source,operation%20on%20the%20underlying%20object.) was just not a thing.

Today, something like 40% of new Node.js projects use Typescript. In fact, this year for the first time NPM Typescript downloads actually [eclipsed Lodash](https://npmtrends.com/lodash-vs-typescript)!

{% include figure image_path="/assets/images/lodash-vs-typescript.png" caption="Lodash vs. Typescript" popup=true %}

Even the remaining Javascript projects depend _heavily_ on type hints delivered into the IDE by dependencies written in Typescript. Speaking for myself, if a library isn't type safe, it's a complete non-starter. I don't care _how_ good it is: I don't want it polluting my project.

Lodash has adapted. Something like 40% of the library has been refactored into TypeScript, and the library is now published as a monorepo with separate packages for each function. This is all good, but it really just masks the REAL problem... which is that Lodash encourages the writing of BAD CODE.

Lodash was always meant to be something of a Swiss Army knife. Most Lodash functions will make a consistent kind of sense out of just about any input type, which is great UNTIL TYPE SAFETY MATTERS! Once you're writing Typescript, you want the exact OPPOSITE behavior. Rather than allow you to write everything and anything, you want your IDE to complain LOUDLY when your code isn't type-consistent from end to end.

Lodash just doesn't do that. In fact, it has the opposite problem: it has tied itself into such knots to be as permissive as possible that you often have to jump through EXTRA hoops just to get a Lodash function to stop producing type errors that an equivalent native Typescript function would never produce.

So: sorry, Lodash guys. Time for a change.

I've experimented with a number of type-safe utility libraries, and at the moment my odds-on favorite is [Radash](https://github.com/sodiray/radash). Radash is written in Typescript, FOR Typescript, and it shows.

One of the interesting things about Radash is what it _doesn't_ do. If you picked up Lodash for the first time this week, you'd see a lot of functions (like `map`, `reduce`, and `forEach`) that are [duplicated by modern Javascript](https://npmtrends.com/lodash-vs-typescript). Radash has none of those. So you don't need to spend cycles deciding what to use: if a function lives in pure JS, Radash wants you to use it.

That's encouraging.

Radash is entirely type-safe. The types that come out of a function are the expected transformations of the types that go in, every time... and they're all Typescript native types. Which makes Radash type-compatible with just about anything.

Moreover, like Lodash, Radash includes a small library of type guards. Unlike the Lodash equivalents , they were INTENDED to be type guards from the start, and they actually WORK.

Radash is also _small_. The whole library weighs in at 306kb, vs. 1.41Mb for Lodash, and the library is MUCH more tree-shakable. So unless you are using the entire library, its effect on your bundle size will be minimal.

Radash is _fast_. It's [not as fast as Lodash](https://www.measurethat.net/Benchmarks/Show/30554/1/lodash-vs-radash-3), but it's fast enough that you won't notice the difference unless you are actually running a benchmark. Did I mention Radash is type-safe?

Finally, the Radash team spent some extra love on the async experience. There are async versions of [`map`](https://radash-docs.vercel.app/docs/async/map) and [`reduce`](https://radash-docs.vercel.app/docs/async/reduce), but my odds-on favorite is [`parallel`](https://radash-docs.vercel.app/docs/async/parallel), which works just like async `map` except that it throttles the number of parallel executions to a specified limit. That's a very nice Lego block to have in the serverless world.

Also, [their documentation](https://radash-docs.vercel.app/docs/getting-started) is pretty. 🤣

If you use Typescript and are still using Lodash: stop. It's time to move on.

Give Radash a try.
