---
# prettier-ignore
title: "dot-code for Windows"
excerpt: "An AutoHotKey script that launches VS Code with the open folder or selected files in File Explorer when you type Alt-Win-."
header:
  og_image: /assets/images/logo-autohotkey.png
  teaser: /assets/images/logo-autohotkey.png
redirect_from:
  - /blog/dot-code-for-windows/
tags:
  - autohotkey
  - projects
  - tips
  - vs-code
  - windows
toc: false
---

<figure class="align-left drop-image">
    <img src="/assets/images/logo-autohotkey.png">
</figure>

This can't just be my problem...

I spend a lot of time bouncing back and forth between different code repos on my machine. I work in Windows, so the quickest way to access all the stuff I want is to keep a File Explorer window open.

So I navigate to where I can see the folder I want to open in VS Code—or maybe just the two files I'm interested in—and then... what?

In principle, I can right-click on the folder and select _Open with Code_ from the context menu. In practice, that option doesn't show up in Windows 11, and I have to select _Show more options_ or hit `Shift-F10` to get there. And then use my mouse to select the menu item.

It's a pain in the butt.

<figure>
  <img src="/assets/images/file-explorer-show-more-options.png">
</figure>

What I really wanted was a keyboard shortcut (I use `Alt-Win-.`, but whatevs) that does three things:

- If I'm in a File Explorer directory and have nothing selected, it opens the directory in VS Code.

- If I have a subdirectory selected, it opens THAT directory in VS Code.

- If I have several things selected, it opens just THOSE things in VS Code.

Fat chance.

I did some searching around, and I found a well-established Windows scripting tool called [AutoHotKey](https://www.autohotkey.com/) (AHK) with plenty of sample code out there.

Trouble is, AHK just released a new major version with a ton of breaking changes, and all the sample code is written in `v1`. Not my jam.

So I took a day, learned AHK `v2`, and [wrote the darn thing myself](https://github.com/karmaniverous/dot-code). Installs in minutes with no config, works like a charm, and my daily frustration level is WAY down!

You're welcome. 😎

{: .text-center}

[See it on GitHub!](https://github.com/karmaniverous/dot-code){: .btn .btn--primary .btn--large}

{: .notice--warning}

**30 days later:** Holy cow I use this thing ALL the time! Just goes to show, you just never know which one of you widgets will wind up being the killer app. Is it getting a lot of play? I dunno. Nor do I CARE, as long as I get to use it lol.
