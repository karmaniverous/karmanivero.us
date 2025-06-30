---
# prettier-ignore
title: "Lodash + Next.js 13 = smh"
excerpt: "Some old carelessness in Lodash plus a stern new outlook in Next.js 13 creates trouble on Edge networks."
header:
  og_image: /assets/images/logo-lodash.png
  teaser: /assets/images/logo-lodash.png
tags:
  - aws
  - javascript
  - logging
  - nextjs
  - troubleshooting
toc: false
---

<figure class="align-left drop-image">
    <img src="/assets/images/logo-lodash.png">
</figure>

I recently updated my [Next.js Template](/nextjs-template) to [Next.js 13](https://nextjs.org/blog/next-13) and have been wrestling with some consequences around middleware interactions with [NextAuth.js](https://next-auth.js.org/).

While troubleshooting this, I added a logging package to my [Next.js middleware](https://nextjs.org/docs/advanced-features/middleware) that uses the [`lodash`](https://www.npmjs.com/package/lodash) [`isPlainObject`](https://github.com/lodash/lodash/blob/master/isPlainObject.js) function. Since that package is articulated using my [NPM Package Template](/npm-package-template), it simply imported the whole Lodash library, since the template has [`babel-plugin-lodash`](https://www.npmjs.com/package/babel-plugin-lodash) in place to cherry-pick any referenced functions.

My [Next.js Template](/nextjs-template) [`dev` environment demo](https://nextjs-template-dev.karmanivero.us) is hosted at [Vercel](https://vercel.com/), which unaccountably does NOT display the logs for my middleware. 🙄 Fortunately I'm using the same template on another project hosted at AWS Amplify, which has better logging (although slower builds), so I added the same logging function to that project, and...

**My build failed!**

<figure> 
  <a href="/assets/images/lodash-nextjs-13-smh-build-failure.png">
    <img src="/assets/images/lodash-nextjs-13-smh-build-failure.png">
  </a>
  <figcaption>AWS Amplify build failure (click to zoom)</figcaption>
</figure>

Most surprising is that the stack trace fingers [`lodash`](https://www.npmjs.com/package/lodash) as the culprit. Hard to imagine a more deeply-vetted package, and my usage of it is about as vanilla as it comes.

Once I did a little digging, I found people yelling about this all over the place. Turns out the [Next.js 13 Edge Runtime](https://nextjs.org/docs/api-reference/edge-runtime#unsupported-apis) constraints are playing havoc with some popular packages.

[This comment](https://github.com/lodash/lodash/issues/5525#issuecomment-1329049661) was particularly interesting, because the author found the culprit in [`lodash`](https://www.npmjs.com/package/lodash)!

[This one line](https://github.com/lodash/lodash/blob/master/.internal/root.js#L11) in the [`lodash`](https://www.npmjs.com/package/lodash) package executes dynamic JavaScript for absolutely no good reason that I can see:

```js
/** Used as a reference to the global object. */
const root =
  freeGlobalThis ||
  freeGlobal ||
  freeSelf ||
  Function("return this")();
```

That's... _disappointing_. Until [`lodash`](https://www.npmjs.com/package/lodash) patches this, a lot of people are going to have to do a lot of workarounds. Including me!

Fortunately, I found a pretty easy one. Most [`lodash`](https://www.npmjs.com/package/lodash) functions are also published as individual packages. I was able to drop the stand-alone [`lodash.isplainobject`](https://www.npmjs.com/package/lodash.isplainobject) dependency into my logger package and my build proceeded without error.

That works for my logging application, but in the more general case I'd be giving up function chaining and other goodies built into the full [`lodash`](https://www.npmjs.com/package/lodash) package.

Meanwhile, I'm subscribed to [this GitHub issue](https://github.com/lodash/lodash/issues/5525) and am tracking it closely!

## Update

The edge-network logging problem turned out to be enough of a pain in the ass that I published my new [`edge-logger`](https://www.npmjs.com/package/@karmaniverous/edge-logger) package to solve it across all my projects.

Give it a shot and let me know what you think!
