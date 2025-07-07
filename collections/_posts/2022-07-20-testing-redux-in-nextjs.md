---
# prettier-ignore
title: "Testing Redux in NextJS"
excerpt: "If you’re testing ES6 code referencing Redux in a transpiled build environment, you can save yourself a lot of trouble by putting your tests into JSX files."
header:
  og_image: /assets/images/logo-redux.png
  teaser: /assets/images/logo-redux.png
redirect_from:
  - /blog/testing-redux-in-nextjs/
tags:
  - nextjs
  - projects
  - react
  - redux
  - testing
  - tips
---

<figure class="align-left drop-image">
    <img src="/assets/images/logo-redux.png">
</figure>

Here's a pretty straightforward problem...

I've got a bunch of NextJS projects in the air that depend on
[this template](https://github.com/karmaniverous/template-nextjs), which uses
the [Redux Toolkit](https://redux-toolkit.js.org/) for state management. I'm
writing ES6 code that is getting transpiled to ES5 during build, and testing it
with [Mocha](https://mochajs.org/) + [Chai](https://www.chaijs.com/).

I've written a lot of tests on both back-end and front-end processes, but none
over Redux. No excuses... I'm pretty new to Redux and have been focused on
learning and using it at the expense of testing it, which is just as dumb as it
sounds. 🙄

Anyway, now I'm making heavy use of Redux Toolkit's
[`createEntityAdapter`](https://redux-toolkit.js.org/api/createEntityAdapter)
and I am _totally_ out of excuses. So yesterday I started writing tests, and I
immediately ran into trouble.

**TL/DR:** If you're testing ES6 code referencing Redux in a transpiled build
environment and have installed
[the new JSX Transform](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html),
you can save yourself a lot of trouble by putting your tests into JSX files. {:
.notice--info}

## Some Weirdness

By the time you read this, I will have incorporated all this stuff into my
template, so for context here's a link to
[the most recent release that doesn't yet address this problem](https://github.com/karmaniverous/template-nextjs/tree/0.0.7).

Some points to note:

- The project is a CommonJS package. It has to be, because on build Babel is
  going to transpile all of our sweet ES6 code down into ES5.

- It's a React project, so there are a lot of JSX files, which thanks to
  [the new JSX Transform](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)
  know how to behave like ES6 code.

- Because we're in CommonJS land, Node will assume that any JS file in the
  project is a CommonJS module, meaning it won't support any ES6 features like
  `import` statements. So if we want to write an ES6 module, we have to put it
  into an MJS file. This includes tests, which of course will want to `import`
  all of our sweet ES6 code.

Now here's something odd. If you look at the
[/state](https://github.com/karmaniverous/template-nextjs/tree/0.0.7/state)
directory, where I keep all my Redux code, you will notice two things:

- I'm performing ES6-style imports.
- The files all have JS extensions.

These are all JS files because they grew up around a bunch of JS sample code.
Nothing ever broke, so I never thought about fixing it.

Weird, right? I still don't understand exactly why that is, but there it is.

## The Setup

In its state as linked above, everything works fine. The project builds, the
front end works, and the one dummy test executes and passes. No problems.

Now let's introduce some entity state.

I don't want this to become a whole disquisition on
[`createEntityAdapter`](https://redux-toolkit.js.org/api/createEntityAdapter),
so just know that this is a layer in the Redux Toolkit that gives you what
amounts to a NoSQL database inside Redux. Super useful. But since you are now
interacting with your Redux Store via this intermediate layer&mdash;which you
need to configure&mdash;it makes sense to add some test support.

I'm going to make three changes in the repo:

1. I'm going to add a new `entitySlice.js` to manage state for a generalized
   entity record that consists of a unique entityKey and timestamps indicating
   when the record was created and last updated. We'll add actions to let us
   add, update, and remove entities, and a selector that lets us fetch an array
   of all entities or just one by entityKey. **Why is this a JS file?** I'm just
   following the weird pattern I already described [above](#the-setup).

2. I'm going to update `store.js` to integrate the new entitySlice into the
   Redux store. **Why is this a JS file?** Same reason.

3. I'm going to write `entity.test.mjs` to exerise all the CRUD operations in
   the new slice. **Why is this an MJS file?** Because it's a Javascript file
   employing ES6 syntax in a CommonJS package. What ELSE should it be?

Here's
[the commit](https://github.com/karmaniverous/template-nextjs/commit/20b7458cbfeb987bb9fbbf28c22565f1dfa3ba4b).
Pretty straightforward, right?

## The Problem

The project built just fine. My first indication I had a problem was when I
tried to TEST it. The Mocha command line just gave up without offering anything
useful, but
[Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter)
offered this in the sidebar:

<figure class="align-center" style="width: 313px">
    <img src="/assets/images/mocha-test-explorer-sidebar-error.png">
</figure>

Doesn't look like much, but clicking on it gave me this:

<figure>
    <a href="/assets/images/mocha-test-explorer-error-output.png"><img src="/assets/images/mocha-test-explorer-error-output.png"></a>
</figure>

Now _that's_ interesting! On the face of it, it looks like
[the weirdness](#some-weirdness) I described above&mdash;where somehow I get to
use ES6 syntax in my Redux-related JS files&mdash;has finally come back to bite
me.

Ok, fair enough. Probably long past time I took care of that anyway. So I
refactored all of my Redux-related JS files to MJS extensions, including any
`include` references to them.

Here's
[the commit](https://github.com/karmaniverous/template-nextjs/commit/b4903eeca1dbbc5e3f022c589b209ac23c8e97c7).
That ought to do it, right?

## The Struggle

Not so fast. The project still builds, but now Mocha shows a different error:

<figure>
    <a href="/assets/images/mocha-test-explorer-error-output-2.png"><img src="/assets/images/mocha-test-explorer-error-output-2.png"></a>
</figure>

Now that just seems perverted. Why is the Redux Toolkit a CommonJS module? Seems
like they would have worked that out by now. Nevertheless, there's a workaround
right in the error message, let's try it out.

I replaced each Redux Toolkit import with the two-step suggested in the error
message. Here's
[the commit](https://github.com/karmaniverous/template-nextjs/commit/bf497acd1a6f566ddb11e22824e58854036b5910).
NOW we're good, right?

No. Not good. Now the BUILD fails! Here's the error:

<figure>
    <a href="/assets/images/mocha-test-explorer-error-output-3.png"><img src="/assets/images/mocha-test-explorer-error-output-3.png"></a>
</figure>

I tried a couple of desperate moves (i.e. `import * as ReduxToolkit from...`)
with similar results. No joy.

I mean Redux is only the single most popular state management system in modern
software development. W the actual F is going _on_ here?

Finally I found
[this comment](https://github.com/reduxjs/redux-toolkit/issues/1960#issuecomment-1021236838)
by [Mark Erikson](https://github.com/markerikson) of the ReduxJS team, dated
this past January:

> Yes, we do not currently ship our package in a fully Node-ESM compatible way.
> (Frankly this entire situation has been incredibly confusing for me every time
> I try to do research on it.)
>
> I've been advised that making any changes to our setup along those lines, such
> as adding an exports field, would be considered a breaking change that
> requires a major version. As such, it's not something we can really even
> consider until whenever we get around to working on an RTK 2.0 version, and
> currently there's no specific timeline for when we'll try to do that.
>
> See [#958](https://github.com/reduxjs/redux-toolkit/issues/958) for some
> related notes.

So it appears that right now the only reliable way to use Redux from ES6 code is
by laundering it through a transpiler. Which, with all due respect to Mark and
team at ReduxJS who are putting out an incredible product for freesies, is NOT
okay.

**Translation:** Not the way I wish software worked! Lord knows I know how hard
this is, and I don't mean to cast any aspersions on the Redux team! They're
doing a bang-up job and for free. Scholars and gentlemen, every one of them. I
just wish this were easier. Now quit punching me, Twitter! 🤣 {:
.notice--warning}

Anyway... what to do?

## The Solution

There's another article I haven't written yet, and it's about testing
[React](https://reactjs.org/) code using [Mocha](https://mochajs.org/) +
[Chai](https://www.chaijs.com/) and the
[React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
in a NextJS project. But the problem I ran into was superficially similar: React
components expressed in JS files got completely different results pre-Babel when
imported by test scrips vs post-Babel in the build process. The error messages
were all on the ES6/CommonJS boundary.

The TL/DR on that one was laughably simple: I just converted all my React
components to JSX files (which they probably should have been to begin with) and
let
[the new JSX Transform](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)
do its thing. At the end, the tests ran and the build succeeded. Problem solved.

So this got me to thinking...

My Redux tests have _nothing_ to do with React. But they _do_ seem to need the
kind of special handling that the JSX transform provides. What if I just
converted all my MJS test files (at least the ones that reference Redux) to JSX
files?

So I did that. Here's
[the commit](https://github.com/karmaniverous/template-nextjs/commit/eba265d1421302c35eec8fec9d64bb2f6e8e73a6).
And the result?

**Works perfectly!** All the tests show up where they belong in the Mocha Test
Explorer sidebar and they run as expected. The project builds and runs as
expected.

But what's the larger lesson here?

**_The map is not the territory._** A JSX file is not a React component, and it
has no obligation to contain one. It's just a bunch of text that will be parsed
in a particular way, according to your development environment's configuration.
If you have a use case where that parsing would be beneficial, then use a JSX
file.

**P.S.** In my rush to get this done and get back to work, I committed my test
file before actually RUNNING the tests. So of course there were some typos.
[This release](https://github.com/karmaniverous/template-nextjs/releases/tag/0.0.8)
ties everything up with a bow, and all the tests actually pass. {:
.notice--info}
