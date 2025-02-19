---
# prettier-ignore
title: "Showing My Code: An Open Response to Elon"
excerpt: 'Elon Musk recently put out a call for coders: "Just show us your code!" Here''s mine.'
header:
  og_image: /assets/images/showing-my-code-banner.jpg
  teaser: /assets/images/showing-my-code-square.jpg
tags:
  - agile
  - design
  - documentation
  - entity-manager
  - github
  - javascript
  - nextjs
  - principles
  - project-governance
  - projects
  - serverless
  - template
  - testing
  - toolkits
  - typescript
toc: false
---

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">If you’re a hardcore software engineer and want to build the everything app, please join us by sending your best work to code@x.com. <br><br>We don’t care where you went to school or even whether you went to school or what “big name” company you worked at. <br><br>Just show us your code.</p>&mdash; Elon Musk (@elonmusk) <a href="https://twitter.com/elonmusk/status/1879531470886465545?ref_src=twsrc%5Etfw">January 15, 2025</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

That was Elon Musk's call to arms on X about 6 weeks ago. I threw my hat into the ring the moment I saw it.
{: .drop-cap}

I'm sure I wasn't alone! Haven't heard back yet, which I arrogantly suppose means Elon just hasn't dug my email out of the pile yet.

So... let's think like a software engineer. Assuming my email is actually worth reading, how could I increase its visibility?

## An Open Response to Elon

Hi there!

I'm responding to [Elon's call for coders on X](https://x.com/elonmusk/status/1879531470886465545).

I am an OLD coder. Been writing software more or less continuously since 1984 (over 4 decades, if you're counting), with a break here & there to accommodate military service. My degree (I know you don't care, but I think it's cool) is in Weapons & Systems Engineering.

My first programming language was [FORTH](<https://en.wikipedia.org/wiki/Forth_(programming_language)>), and I would have to count on my fingers to tell you how many I've learned. Dozens, for sure, if you count config & markup. Picking up a new one at this point is meh.

These days I am a full-stack developer. I prefer Node.js on the back end, Next.js on the front end, and Typescript on both. I'm flexible, but I will admit that I have NSFW feelings about type safety.

The last several years of my GitHub profile look like this because I am VERY good at writing DevOps automation:

{% include figure popup=true image_path="/assets/images/2025-02-10-contributions.png" caption="GitHub contributions in the past year." %}

Building tools that empower developers (including myself) is my truly happy place.

I'm a big believer in [sound engineering principles](/mixin-it-up-picking-the-right-problem-to-solve/), [design on purpose](/toolkits/project-governance/turning-the-crank-design-as-a-mechanical-process/), and [Agile as a living process](/toolkits/project-governance/a-modern-agile-project-manifesto/). I'm obsessive about documentation. I write unit tests because this forces me to write testable code, and testable code is BETTER code.

I know what GOOD looks like.

Here are some of my recent public projects:

- [controlled-proxy](https://github.com/karmaniverous/controlled-proxy) is a very lightweight, universal dependency injector implemented in Typescript.

- [entity-manager](https://github.com/karmaniverous/entity-manager) implements rational indexing & cross-shard querying at scale in your NoSQL database so you can focus on your application logic. Still a work in progress, tons of documentation [here](https://karmanivero.us/projects/entity-manager/intro/).

- [jsonmap](https://github.com/karmaniverous/jsonmap) is a hyper-generic JSON mapping library.

- [metastructure](https://github.com/karmaniverous/metastructure) is a config-driven, enterprise-grade, open-source application infrastructure generator that keeps your Terraform DRY as a bone. Documentation wiki [here](https://github.com/karmaniverous/metastructure/wiki).

- [mock-db](https://github.com/karmaniverous/mock-db) mocks DynamoDB-style query & scan behavior with local JSON data.

- [serify-deserify](https://github.com/karmaniverous/serify-deserify) reversibly transforms unserializable values into serializable ones. Includes Redux middleware.

- [serverless-lodash-plugin](https://github.com/karmaniverous/serverless-lodash-plugin) enables variable expressions in serverless.yml using the full Lodash feature set, plus some extra goodies!

These are mostly back-end stuff. My most significant recent full-stack project is live at [VeteranCrowd](https://veterancrowd.com), but you'll have to ignore the ugly public home page and create an account to have the full experience! This is a VERY involved FinTech application that integrates a ton of third-party functionality, happy to do a walkthrough if we get that far.

[Karmic Rules for Writing Pretty Good Code](https://github.com/karmaniverous/rules/) #3: _If you want to get a complex process right, template it._ I maintain this highly functional [Typescript NPM Package Template](https://github.com/karmaniverous/npm-package-template-ts), which some have found useful.

If you still haven't seen enough, visit [my blog](/) which is unabashedly technical. Jenkins, if it matters.

I think what you guys are doing is great and I'd love to be a part of it.

Regards,

Jason Williscroft
