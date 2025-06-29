---
# prettier-ignore
title: "Directing TEL Links to WhatsApp Desktop in Windows"
excerpt: "Wish you could click on a phone link in Windows and open it in your WhatsApp app? Here's how!"
header:
  og_image: /assets/images/dual-whatsapp-banner.jpg
  teaser: /assets/images/dual-whatsapp-square.jpg
tags:
  - tips
  - whatsapp
  - windows
---

<figure class="align-left drop-image">
    <img src="/assets/images/dual-whatsapp-square.jpg">
</figure>

Here in Indonesia, WhatsApp is the go-to messaging service. SMS is for spam. WhatsApp is for actually getting things done.

I have WhatsApp installed on my Windows desktop, because why not. And when I manage employees in an HR app or candidates on a job board, I want to be able to click on their phone numbers and either start or resume a conversation with them in WhatsApp.

Since I spend so much time in front of my laptop, I make extensive use of the WhatsApp desktop client. Just makes sense.

**TL/DR:** In Windows 11, go to **Default apps** in your System Settings and set the default app for `TEL` and `WHATSAPP` links to be your WhatsApp app. _Pro tip: [install the WhatsApp Beta app](/using-two-whatsapp-accounts-on-the-same-desktop/) to use a second WhatsApp account on your desktop!_
{: .notice--info}

This will be child's play for you other developers out there, but my mom isn't a developer, so this is for her. 💖

## What's A TEL Link?

Let's back up: what's a _link?_

You see them all the time: links are those underlined words or phrases you click on to open a web page. Under the hood, those links look something like this:

```
https://karmanivero.us/directing-tel-links-to-whatsapp-desktop-in-windows
```

The `https:` part is the link's _protocol_. It describes the type of the link, so your computer knows what to do with it. In this case, your computer will open a secure link in a web browser.

There are many other protocols, most of which you've likely seen and used, even if you didn't know it. For example:

| Protocol    | Description                                             |
| ----------- | ------------------------------------------------------- |
| `http:`     | Opens an unsecured web page in your browser             |
| `ftp:`      | Opens a file transfer protocol client to download files |
| `file:`     | Opens a file on your local system                       |
| `mailto:`   | Opens your email client to send an email                |
| `tel:`      | Opens your phone app to make a call                     |
| `whatsapp:` | Opens WhatsApp to start a chat                          |

More often than not, when you see a phone number linked on a web page, it's a `tel:` link. Under the hood, the link will look something like this:

```
tel:+12345678900
```

When you click on it, your computer will probably do _something..._ most likely open your phone app and start a call to that number.

In principle, links using the `whatsapp:` protocol should open WhatsApp and start a chat with that number. In practice, that protocol isn't used very much... and even when it is, your system might not know what to do with it. We'll handle both of these cases in this post.

## Install WhatsApp Desktop

Before we can set up WhatsApp as the default app for `tel:` links, we need to make sure you have WhatsApp installed on your Windows desktop.

That's easy: just go to the Microsoft Store and download the [WhatsApp app](https://apps.microsoft.com/detail/9nksqgp7f2nh). If you want to use two WhatsApp accounts on the same desktop, you can also [install the WhatsApp Beta app](/using-two-whatsapp-accounts-on-the-same-desktop/) and attach it to a different account!

## Set Default Apps

The link protocol tells your system what kind of link it is. That isn't the same as knowing what to _do_ with that kind of link. That's a matter for your System Settings.

To get there, open your Windows Settings (try `Win-I`), type "link" in the search box, and select **Choose a default app for each link type**.

{% include figure popup=true image_path="/assets/images/whatsapp-tel-1.png" alt="Choose a default app for each link type." caption="*Choose a default app for each link type.*" %}

Once you get there, search or scroll down to the `TEL` and `WHATSAPP` link types. Click on each one, choose WhatsApp from the list of apps, and click the **Set default** button. That's it! Now when you click on a phone number (or a WhatsApp link, which are rare) in a web page or just about anywhere else on your machine, it will open in WhatsApp!

{% include figure popup=true image_path="/assets/images/whatsapp-tel-2.png" alt="Set the default link handler." caption="*Set the default link handler.*" %}

## Some Gotchas

Not every phone number is on WhatsApp, and your local machine has no way of distinguishing those that are from those that aren't. So if you click on a phone number that isn't associated with a WhatsApp account, it will STILL open up in WhatsApp! It's up to you to figure out what to do next.

Also, if you followed [these instructions](/using-two-whatsapp-accounts-on-the-same-desktop/) to set up a second WhatsApp account on your desktop, you now have two WhatsApp apps running on your machine—WhatsApp and WhatsApp Beta—each connected to a different account. When you set up your `tel:` and `whatsapp:` links as described above, you're going to have to choose one of them as the default link handler. Just pick the one attached to the account you use most.

You're welcome. 😎

I'm not a Mac user, but **I bet there's a similar solution for MacOS.** If you know what it is, please share it in the comments!
{: .notice--info}
