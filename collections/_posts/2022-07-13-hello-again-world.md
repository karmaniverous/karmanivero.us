---
# prettier-ignore
title: "Hello (Again) World!"
excerpt: "You spend years of perfectly good writing time building gilded prisons for your words. If only there were a simple, beautiful way to write PORTABLE words."
header:
  og_image: /assets/images/jekyll-square.png
  teaser: /assets/images/jekyll-square.png
redirect_from:
  - /blog/hello-again-world/
tags:
  - jekyll
  - writing
toc: false
---

<figure class="align-left drop-image">
    <img src="/assets/images/jekyll-square.png">
</figure>

I can't remember how many blogs I've had.

Different times, different collaborators, different platforms. That last is a
bitch, because try pulling three years of work out of a decades-old
[DotNetNuke](https://www.dnnsoftware.com/) database backup to migrate to
WordPress.

It's on a Zip drive someplace. 🙄

So the platform is the thing, almost more than the words. You want writing to be
easy and pleasant, so you spend years of perfectly good writing time building
gilded prisons for your words. If only there were a simple, beautiful way to
write _portable_ words.

Well.

I feel pretty confident about my GitHub repos. Definitely mine. And I love
writing in Markdown. And I _love_ the idea of sticking a bunch of words into a
[GitHub Pages](https://pages.github.com/) repo and having them just... _appear_.
Oh glory day.

So yesterday I spent about a thousand hours trying to get
[Jekyll + Pages](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll)
to work. It was AMAZING, right until I tried to add content or a theme. Then...

I admit I am not a Ruby guy. Still. Very hard. But then I (finally) had the
brilliant idea to search for the
[most-forked Jekyll theme repos](https://github.com/search?o=desc&q=jekyll+theme&s=forks&type=Repositories),
and right at the top I found
[this one](https://github.com/mmistakes/minimal-mistakes). Meaning THIS one.

Problem solved.

My suggestion: take the easy road from the
[Quick-Start Guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)
and create your repo from the
[Minimal Mistakes template](https://github.com/new?template_name=mm-github-pages-starter&template_owner=mmistakes).
You can be up and running in single-digit minutes. It just WORKS.

I only had one issue, which turned out to have an easy fix.

Between configuring your site and drafting content, you'll want to run Jekyll
locally so you can see your results before you push to GitHub and wait for the
build process to complete.

In principle, this should be easy:

1. Install Ruby, bundler, and Jekyll. See steps 1 & 2
   [here](https://jekyllrb.com/docs/#instructions).
2. Pull your repo.
3. Run `bundle install` to pull all your dependencies.
4. Run `bundle exec jekyll serve` to start your local Jekyll server.
5. Visit your local site at [http://127.0.0.1:4000](http://127.0.0.1:4000)

Only it didn't work.

I got two errors (with some stuff in the middle I pulled out for clarity):

```
GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.

cannot load such file -- webrick
```

I found a reference to the first error
[here](https://github.com/github/pages-gem/issues/399). TL/DR: add this line to
the bottom of your `_config.yml` file:

```yml
github: [metadata]
```

I found the second one solved
[here](https://talk.jekyllrb.com/t/load-error-cannot-load-such-file-webrick/5417/2).
TL/DR: run `gem install webrick` and then add this line to the bottom of your
`Gemfile`:

```ruby
gem "webrick"
```

You might need to run `bundle install` again, not sure. Won't hurt.

Anyway, after that you should be able to run `bundle exec jekyll serve`, watch
your site build, and see it locally at
[http://127.0.0.1:4000](http://127.0.0.1:4000). Make a content change, save it,
and your pages will rebuild instantly!

_Very_ pretty theme, totally configurable. And your content is _text_. Doesn't
get more portable than that.

I'm sold.
