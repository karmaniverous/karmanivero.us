---
# prettier-ignore
title: "Vercel Preview Branch Weirdness"
excerpt: "An accidental hosting environment dependency revealed an irritating bug in Vercel's preview deployment logic."
header:
  og_image: /assets/images/logo-vercel.png
  teaser: /assets/images/logo-vercel.png
tags:
  - javascript
  - nextjs
  - troubleshooting
toc: true
---

<figure class="align-left drop-image">
    <img src="/assets/images/logo-vercel.png">
</figure>

I've got a very handy
[Next.js project template](https://github.com/karmaniverous/nextjs-template) out
there that I am using as the basis of a number of projects.

I host all of these projects on [Vercel](https://vercel.com/), because srsly why
wouldn't I. So this morning I pulled one of those low-priority, high-irritation
issues off my backlog and discovered two things:

- My template has a dependency on the Vercel hosting environment, as opposed to
  the Next.js platform.
- Vercel's preview domains don't exactly work as advertised.

I'll fix the template, eventually. But meanwhile, this Vercel thing is
interesting.

## The Setup

If you are reading this, you are probably familiar with the
`process.env.environment` variable. It's what Node.js uses to identify your
runtime environment. In Next.js—which is built on top of Node.js—this is either
set to `development` or `production`.

Vercel introduces the concept of a _preview environment_. There are lots of ways
to use Vercel, but probably the most common is to connect your Vercel account to
GitHub and designate a project branch (_e.g._ `main`) as your production branch.
Vercel creates a build when you deploy to _any_ GitHub branch and attaches it to
a randomized domain. A deployment anywhere but your designated production branch
is a preview deployment. You can read this state on the front end at
`process.env.NEXT_PUBLIC_VERCEL_ENV`.

If you're looking at `process.env.NEXT_PUBLIC_VERCEL_ENV` on a published branch,
its value will either be `production` or `preview`. If you're looking for it in
your development environment, it won't be set—think about it—unless you set it
explicitly from an environment file using something like
[`dotenv`](https://www.npmjs.com/package/dotenv).

So my Next.js template has a Coming Soon page. To make it work, I have entries
in two environment files:

`.env.development`

```sh
# Mimic runtime environment in dev.
NEXT_PUBLIC_VERCEL_ENV=development

# Show/hide coming soon page.
NEXT_PUBLIC_COMING_SOON=0
```

`.env.production`

```sh
# Show/hide coming soon page.
NEXT_PUBLIC_COMING_SOON=1
```

Then there's logic
[here](https://github.com/karmaniverous/template-nextjs/blob/4b81be555ec7be25ede64c081010da86d460f1b9/pages/_app.jsx#L25-L27)
and
[here](https://github.com/karmaniverous/template-nextjs/blob/4b81be555ec7be25ede64c081010da86d460f1b9/pages/_app.jsx#L210-L214)
on my `_app.jsx` page that redirects either toward or away from the Coming Soon
page, based on these settings:

```jsx
const isComingSoon =
  process.env.NEXT_PUBLIC_COMING_SOON === "1" &&
  process.env.NEXT_PUBLIC_VERCEL_ENV !== "preview";

// If app is coming soon, route all traffic to coming-soon endpoint.
if (route !== "/coming-soon" && isComingSoon)
  redirect(res, "/coming-soon");

// If app is not coming soon, route all coming-soon traffic to home.
if (route === "/coming-soon" && !isComingSoon) redirect(res, "/");
```

Get it? I can set `NEXT_PUBLIC_COMING_SOON` to make the Coming Soon page appear
(or not) in `development` or `production`, but it _never_ appears in `preview`,
because **_preview_**. I'm sure there is a way to do this without the dependency
on a Vercel-specific environment variable, but so far I haven't _needed_ one.

## The Problem

As I mentioned above, every time you push to a GitHub Branch, Vercel builds your
project and deploys it to a unique URL. Two, actually: one is always generated,
and another is specific to the attached GitHub branch. For example, if I pushed
application `myapp` to GitHub branch `preview-0-3-0`, the application would
deploy to domains like these:

- `myapp-9z3i9bqtd-myaccount.vercel.app`
- `myapp-git-preview-0-3-0-myaccount.vercel.app`

This is handy when you want to publish links to preview versions of your
application, which—if you use my template—will _never_ display the Coming Soon
page.

But I thought I'd be cute.

If you've got a domain (_e.g._ `mydomain.com`) attached to your Vercel project,
Vercel also offers the ability to attach a custom subdomain to your preview
branch. So, following the example above, I could create
`preview-0-3-0.mydomain.com`.

Nice, right? The configuration looks like this:

<figure>
    <a href="/assets/images/vercel-preview-branch.png"><img src="/assets/images/vercel-preview-branch.png"></a>
</figure>

Vercel's intent with this is clear. Nevertheless, when I visit my new subdomain,
what do I see? _The Coming Soon page!_

Yet when I visit either of the automatically-generated domains, I see the
application home page, as expected.

Pretty obvious what is going on here: despite the clear indication that my
custom domain is a `preview` deployment, Vercel has set the
`NEXT_PUBLIC_VERCEL_ENV` environment variable to `production`.

😐 {: style="font-size: 3em; text-align:center;"}

## The Solution (Kind Of)

I'm not going to go very far out of my way to fix this.

Instead, I've opened
[a ticket](https://github.com/vercel/vercel/discussions/8340) with Vercel and
hope they will resolve the issue soon. Meanwhile, I've altered all my preview
deployment links to point to the (way less sexy) automatically generated
domains.

There is still the question of the Vercel dependency in my template. I intend to
fix this (here's
[my own ticket](https://github.com/karmaniverous/template-nextjs/issues/20)) but
I do have some larger fish to fry.

If you want to use my template but the Vercel dependency is a problem for you,
feel free to drop a comment below and I'll bump it up in my queue.

Or, you know, roll up your sleeves and fix it for me!
[#opensource](https://twitter.com/hashtag/openSource)
