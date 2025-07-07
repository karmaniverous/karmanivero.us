---
# prettier-ignore
title: "Unifying Web Presence with GitHub Pages"
excerpt: "Eliminate content collisions with your primary GitHub Pages domain & unify all your GitHub project documentation pages under a single subdomain."
header:
  og_image: /assets/images/unify-web-presence-banner.jpg
  teaser: /assets/images/unify-web-presence-square.jpg
redirect_from:
  - /blog/unifying-web-presence-with-github-pages/
tags:
  - documentation
  - github
  - jekyll
  - writing
---

<figure class="align-left drop-image">
    <img src="/assets/images/unify-web-presence-square.jpg">
</figure>

A couple of years ago I shifted all of my personal web publishing (blogging, portfolio, etc.) to [GitHub Pages](https://pages.github.com/). I've written [here](/blog/hello-again-world), and more recently [here](/blog/hello-world-redux), about how & why I did that.

This week I faced a different but related challenge. The solution turned out super satisfactory and super simple, so I thought I'd share.

**TL/DR:** Use your documentation subdomain as your _primary_ GitHub Pages domain, and you can publish docs from all your repos under a single subdomain while avoiding content collisions with your main domain.
{: .notice--info}

## The Problem

For the sake of this discussion, remember my GitHub user name is `karmaniverous`.

A typical GitHub Pages user will have a public personal repository called something like `karmaniverous.github.io`. This repository is the root of their GitHub Pages site. The usual configuration is to serve content from this repo's `main` branch, and to map the default domain assigned to the repo (e.g. `karmaniverous.github.io`) to a custom domain (e.g. `karmanivero.us`).

There will likely be some sort of a [Jekyll](https://jekyllrb.com/) theme in the repo, but for this discussion it doesn't matter: there's _content_. And when you point your browser at `https://karmanivero.us`, you see the content.

Easy peasy.

Lately, I've been doing a lot of work in TypeScript, and I've been using [TypeDoc](https://typedoc.org/) to generate API documentation on my projects. This documentation lands in the `docs` directory of the respective repository. I can go to the Pages section of my `foo` repo settings and tell GitHub to publish the content in that directory to my Pages domain. Consequently, the documentation shows up at `https://karmaniverous.com/foo`.

Whatever I do in the `foo` repo is completely independent of what I doing in the `karmaniverous.github.io` repo:

- `karmanivero.us/portfolio` is a page on my blog, served from the `karmaniverous.github.io` repo.

- `karmanivero.us/foo` is the API documentation for the `foo` project, and is served from the `foo` repo.

**What if I want to add a `foo` page to my blog?**

This isn't as ridiculous as it sounds. There's a lot more to the `foo` project than just the API documentation, so it really makes sense to create a `foo` page on my blog to pull all that stuff together under the obvious heading. Only... that page already exists!

So how do I make this work?

## Custom Subdomains?

When I assigned the `karmanivero.us` domain to my `karmaniverous.github.io` repo, I set it as the default Pages domain for _all_ of my GitHub repos. That's why I didn't have to do anything special to get my `foo` API documentation to display at `karmanivero.us/foo`. As soon as I enabled Pages on the `foo` repo, the content just showed up.

But there's nothing preventing me from assigning a _different_ custom domain to the `foo` repo!

So I created `docs.karmanivero.us` and gave it a CNAME to `karmaniverous.github.io`. See the [GitHub docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site) for more info on that.

Then I assigned `docs.karmanivero.us` to the `foo` repo instead of `karmanivero.us`, and...

Well.

My _expectation_ was that the `foo` API documentation would show up at `https://docs.karmanivero.us/foo`. But it didn't. Instead, it showed up at the base subdomain, at `https://docs.karmanivero.us`.

If I only ever had one repository with API documentation, I guess it wouldn't matter. But in fact there's also the `bar` project, and I was hoping I could make _its_ API documentation show up at `https://docs.karmanivero.us/bar`. But clearly this approach is going to send both sets of documentation to the same place.

In practice, I didn't even get _that_ far... once I had assigned `docs.karmanivero.us` to the `foo` repo, GitHub wouldn't even allow me to assign the same subdomain to the `bar` repo.

I briefly considered this hack: create custom subdomains `foo` and `bar`, and then assign `foo.karmanivero.us` and `bar.karmanivero.us` to the `foo` and `bar` repos, respectively! And that _would_ work... but then I'm stuck creating a new subdomain for every project I want to document. That's going to get old fast.

What I really want is to have a _single_ subdomain like `docs` where _all_ of my project documentation lands, but whose content (because it's on a subdomain) will _never_ collide with content on my main domain.

## The Solution

This is really a classic case of turning the problem on its head.

When I only had a single GitHub Pages repo (my blog at `karmaniverous.github.io`), it made perfect sense to assign my root domain `karmanivero.us` to it. The fact that this became my default domain for _all_ Pages-enabled repos was neither here nor there, because I didn't have any others.

As I worked through the problem above, that configuration was my starting point. As a consequence, each of the repos requiring a documentation path under the `docs` subdomain became an odd man out _in exactly the same way!_.

So here's what I did:

- I renamed the `karmaniverous.github.io` repo to `blog`. It still has Pages enabled, and the `karmanivero.us` domain is still assigned to it.

- I created a _new_ repo called `karmaniverous.github.io`, enabled Pages, and assigned subdomain `docs.karmanivero.us` to it. Ths is now the default Pages domain for all of my repos (unless I assign a different custom domain).

- To keep things very clean, I added a redirect in the new `karmaniverous.github.io` root so that anybody who tries to visit `https://docs.karmanivero.us` will instead land on my main site at `https://karmanivero.us`.

Now...

- All my blog content shows up under `https://karmanivero.us`

- All my `foo` and `bar` project API documentation shows up under `https://docs.karmanivero.us/foo` and `https://docs.karmanivero.us/bar`, respectively.

- I don't have to do _anything_ to make it work except enable Pages on a new project repo!

I set the documentation root up as a [repo template](https://github.com/karmaniverous/karmaniverous.github.io), so if you like you can easily spawn a copy & change the redirect to point to your own root domain.

I hope you find this useful!
