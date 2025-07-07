---
# prettier-ignore
title: "TSDoc + TypeDoc"
excerpt: "Mastering the obvious, one lexical standard at a time."
header:
  og_image: /assets/images/tsdoc-plus-typedoc.png
  teaser: /assets/images/tsdoc-plus-typedoc.png
redirect_from:
  - /blog/tsdoc-plus-typedoc/
tags:
  - ai
  - documentation
  - template
  - typescript
toc: false
---

<figure class="align-left drop-image">
    <img src="/assets/images/tsdoc-plus-typedoc.png">
</figure>

Today left me feeling pretty sure everybody else knows some stuff I don't.

It all started with the [`@category`](https://typedoc.org/tags/category/) tag. See, I want to use it with [TypeDoc](https://typedoc.org) to generate sidebar navigation categories in API documentation pages like [this one](https://docs.karmanivero.us/string-utilities/). Which is awesome, by the way, and built into my [Typescript NPM Package Template](https://github.com/karmaniverous/npm-package-template-ts).

And it's also less of a shameless plug than it sounds, because (a) I use that template a LOT, and (b) the freaking `@category` tag was throwing a linting error. Like, _everywhere_.

**TL/DR:** Add [this file](https://github.com/TypeStrong/typedoc/blob/master/tsdoc.json) to your package root to get [`eslint-plugin-tsdoc`](https://www.npmjs.com/package/eslint-plugin-tsdoc) to support [TypeDoc](https://typedoc.org) right out of the box!
{: .notice--info}

The source of the error was [`eslint-plugin-tsdoc`](https://www.npmjs.com/package/eslint-plugin-tsdoc), which stinks becuse that ESLint plugin has exactly ONE rule. So it's either on or off. And I _really_ want to lint my TSDoc comments. But TypeDoc defines a few _extra_ tags—like `@category`—that `eslint-plugin-tsdoc` doesn't know about. So I was stuck.

Google was no help at all.

So I logged onto ChatGPT, and before long I'd talked myself into writing a wrapper around `eslint-plugin-tsdoc` to catch & ignore errors on valid TypeDoc tags. And, you know, in GPT-land ANYTHING is possible!

That ate up a couple of hours. Let's just say I'm not proud of it.

Then I thought I'd just fork `eslint-plugin-tsdoc` and add the extra tags myself. And THAT was when I finally discovered [`@microsoft/tsdoc-config`](https://tsdoc.org/pages/packages/tsdoc-config/), a library that loads custom TSDoc configs from a `tsdoc.json` file. Which, it turns out, is a dependency of `eslint-plugin-tsdoc`.

Translation: if you're using `eslint-plugin-tsdoc`, it is ALREADY looking for a `tsdoc.json` defining custom tags!

And when I went back to the [TypeDoc repo](https://github.com/TypeStrong/typedoc), what do you think I found? Oh yah: a big fat [`tsdoc.json`](https://github.com/TypeStrong/typedoc/blob/master/tsdoc.json), all loaded up with custom TypeDoc tags. Including `@category`.

So I copied that `tsdoc.json` into my project, and the `@category` linting error went away. Ditto for my template.

Maybe Google will be kinder to the next guy.
