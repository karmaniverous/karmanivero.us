---
# prettier-ignore
title: "When IntelliSense Fails: How to Avoid TypeScript While Avoiding TypeScript"
excerpt: "Sure you could transpile perfectly good ES6 code into the same thing + TS types. But sweet mother of everything, WHY??"
header:
  og_image: /assets/images/logo-vscode.png
  teaser: /assets/images/logo-vscode.png
redirect_from:
  - /blog/when-intellisense-fails/
tags:
  - javascript
  - tips
  - typescript
  - vs-code
toc: false
---

<figure class="align-left drop-image">
    <img src="/assets/images/logo-vscode.png">
</figure>

The handful of people who actually pay attention to these words may be aware that I maintain a growing list of small GitHub projects that do useful things.

Not trying to be famous; this is all stuff I use in my own projects, and I've long since learned that the best way to robustify your code is to let other people see it. See [my GitHub profile](https://github.com/karmaniverous) for more info.

So this morning I imported one of my packages into an ES6 project and hit this weird WTF moment: its IntelliSense was broken! The imported function ran just fine, but showed absolutely nothing at the tooltip. All that careful [JSDoc](https://jsdoc.app/): worthless.

Strangely, IntelliSense worked just fine when I altered my import to point at a specific file in `node_modules`. Although then the function failed to execute because the specific file wasn't a designated export of that module.

**Smells like a clue, right?**

So I started investigating and immediately ran into a few articles and StackOverflow responses like [this one](https://stackoverflow.com/a/74012034/17920604). TL/DR: helpful TypeScript users thoughtfully explain how to augment perfectly functional ES6 code with TypeScript types in order to get VS Code to to the job it ALREADY has enough info to do!

This is the sort of thing that could have me drinking tequila at 10 in the morning.

**I rebel!**

I am writing JavaScript. I do NOT want to transpile my very sweet ES6 code, already thoughtfully decorated with JSDoc, into the EXACT SAME THING just so my code can be augmented with types in a language I am studiously trying to avoid.

That's coo-coo.

As far as I am concerned, **my ES6 code IS my ES6 distribution!** I want to point my `package.json` `exports['.'].import` key at my `lib` directory (where my source lives) and have done with it. This should just WORK, and if it doesn't, then the problem lies with the TOOL and not with my code.

So.

It turns out that doing what I just described, works JUST FINE. All your imports work, all the code runs, all that. The only failure is that IntelliSense breaks in VS CODE.

How come?

Well, clearly VS Code is parsing `package.json` to figure out where your typedefs are, and apparently **VS Code has not yet caught up to the new export-mapping syntax!**

Take a look at the Node.js [package entry points](https://nodejs.org/api/packages.html#package-entry-points) documentation. The `main` key still works, and if you use it alongside `exports`, Node.js will just ignore it. But VS Code will pick it up and use it to power your IntelliSense!

For example:

```json
{
  ...,
  "exports": {
    ".": {
      "import": "./lib/index.js", // where I write my code
      "require": "./dist/default/lib/index.js" // Babel transpiler target
    }
  },
  "main": "./lib/index.js", // where VS code looks for IntelliSense
  ...
}
```

This works perfectly! The imported module will expose all your JSDoc to IntelliSense, and now we can all go back to earning a living.

Again, no aspersions cast on you TS-niks! You guys are awesome. I just don't want to be one of you. 👊

**P.S. How did this manage to catch me by surprise?** A few months ago I refactored my templates to use the new `exports` key and neglected to test the effect of the change on package imports.

You may point and laugh at your discretion. 🙄
