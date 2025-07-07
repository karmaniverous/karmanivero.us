---
# prettier-ignore
title: "Related Posts With Minimal Mistakes"
excerpt: "Presenting a radically improved related-posts widget for the Minimal Mistakes Jekyll theme."
header:
  og_image: /assets/images/related-posts-banner.jpg
  teaser: /assets/images/related-posts-square.jpg
redirect_from:
  - /blog/related-posts-with-minimal-mistakes/
tags:
  - jekyll
---

<figure class="align-left drop-image">
    <img src="/assets/images/related-posts-square.jpg">
</figure>

As I publish more and more content, I am increasingly concerned about connecting the dots between the post you are now reading and other, related posts. Just like every other blog in the world, I use a _Related Posts_ section at the bottom of each page for that purpose.

**TL/DR:** See the list of related posts at the bottom of this page. That's what we're building here.
{: .notice--info}

## Jekyll in a Nutshell

This site is a [Jekyll](https://jekyllrb.com/) instance, based on the very excellent [Minimal Mistakes](https://github.com/mmistakes/minimal-mistakes) theme and hosted on [GitHub Pages](https://pages.github.com/). As a matter of expediency, I'm using the [remote version](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#remote-theme-method) of the theme, which reduces the complexity a bit but limits the Jekyll plugins I can use.

Content items in Jekyll break down into two broad categories: [pages](https://jekyllrb.com/docs/pages/) and [posts](https://jekyllrb.com/docs/posts/). Pages lay out the broad structure of the site (like your About or Portfolio page, although your Home page can be a special case). Posts are the individual articles you write. Think of a post as _dated_ content, while a page is _evergreen_ content.

Posts are grouped into [collections](https://jekyllrb.com/docs/collections/), each under its own root path. For example, on this site, all blog posts are under the [`/blog`](/blog) path, and the Entity Manager documentation is collected under the [`/projects/entity-manager`](/projects/entity-manager) path. Each collection also defines its own taxonomic & layout defaults, so it's easy to create a common look and feel for all the posts in a collection.

Jekyll supports the assignment of [tags and categories](https://jekyllrb.com/docs/posts/#tags-and-categories) to any post or page. Most themes have special layouts that leverage these taxonomies to create tag and category indexes like [this one](/topics).

Every post or page can have [front matter](https://jekyllrb.com/docs/front-matter/). This is a block of YAML at the top of the file that defines its tags, categories, title, layout, and other metadata, and also sets variables that can be consumed by the theme to do things like toggling the display of a table of contents or—today's topic—**related content**.

## Related Content: What Good Looks Like

So when you look for related content at the bottom of a post or page, what should you actually see? Here's a list of requirements:

- The related content should be **relevant** to the post or page you are reading. We ought to be able to leverage common tags to determine relevance (let's leave categories out of this discussion; the principle is the same).

- The related content should be **prioritized**. Content that is newer and/or more relevant should appear higher in the list.

- The related content should be **limited**. I should only see the top few _most_ relevant items, not every post that shares a tag with the one I'm reading.

- The related content should be **type-independent**. A page or a post should be able to relate to a another page or a post in a different collection if they have at least one tag in common.

- The related content should be **rational**. I shouldn't see hidden posts. Unless this page is a blog post, I shouldn't see other posts from the same collection. If a post redirects to a page, the redirecting post shouldn't appear in the list... but if the redirecting post has tags assigned to it, then they should factor into the redirection target's relevance score!

## What's Actually In The Box

The good news is that the Minimal Mistakes theme actually has a [related posts include](https://github.com/mmistakes/minimal-mistakes/blob/master/_includes/page__related.html) built in. If you set `related_posts: true` in your post's front matter, the include will generate a list of related posts at the bottom of the rendered page.

The bad news is that this widget is pretty basic. All it _really_ does is display the most recent posts in your timeline: the same ones on every page, with no respect to actual relevance.

To the author's credit, he's aware of the gap. There's a [closed issue](https://github.com/mmistakes/minimal-mistakes/issues/554) in the Minimal Mistakes repo that addresses the problem, and the author published [a solution](https://github.com/mmistakes/minimal-mistakes/issues/554#issuecomment-302143467) that addresses it.

But...

- This solution doesn't do any of the relevance scoring or sorting described above, and it won't be included in the official theme due to the author's concerns about build performance. Which is fair.

- Relevant posts in this solution are all constrained to the posts collection. It won't show pages or posts from other collections, and it also won't filter out redirected posts.

So probably not the solution I'm looking for.

**This file is a [Liquid](https://shopify.github.io/liquid/) include!** So anything we build here will be a drop-in replacement for the existing include, also implemented in Liquid.
{: .notice--info}

Let's get to work.

## An Improved Alternative

[This post](https://blog.webjeda.com/jekyll-related-posts/) (H/T [Fabrizio Musacchio](https://www.fabriziomusacchio.com/) and his excellent [Minimal Mistakes Cheat Sheet](https://www.fabriziomusacchio.com/blog/2021-08-11-Minimal_Mistakes_Cheat_Sheet/)) outlines a strategy for scoring related posts based on the number of tags they have in common.

It's an improvement over the Minimal Mistakes solution, but...

- it isn't specific to the Minimal Mistakes theme,

- it still doesn't support pages or non-blog posts, and

- it doesn't factor in recency or redirects.

But it got my creative juices flowing. So yesterday I rolled up my sleeves and went to work.

## Calculating Relevance

Let's say that there are two factors to calculating relevance $$R$$:

- The number of tags $$N$$ a given post has in common with the current page.

- The age in days $$A$$ of the post.

Let's take the log of both of these values, since the difference between today's post and yesterday's post should count for more than that between a year-old post and one aged a year and a day. Same argument applies for common tag counts.

Let's use a base-2 log as it will be a little easier to implement in Liquid.

Let's weight the contribution of $$N$$ by $$W_{tags}$$, and normalize $$N$$ by the maximum number $$N_{max}$$ of common tags across all posts so we don't need to change $$W_{tags}$$ as we add new tags to our system.

Then we have:

$$R = W_{tags} \ log_2 \dfrac{N}{N_{max}} - log_2 A$$

Pages don't have a post date, so calculating their age is problematic. There are plugins that will get the last-modified date of a file, but...

- None of them are white-listed for GitHub Pages, so they won't work with the remote version of the Minimal Mistakes theme.

- None of them perform very well anyway.

- Pages are generally evergreen content, so they _should_ be treated as fairly recent, regardless when last modified.

Easy solution: let's give all pages a default age of 32 days. That way they'll have a high calculated relevance, but recent posts can still compete with them on tag relevance.

As calculated above, $$R$$ will have a small value in the one-to-two-digit range, centered around zero. To get it to work with a text-based Liquid sort, we'll need to scale it up, add a big number, and round it off so it has a constant width. We'll handle that in the code.

## Liquid Logs

Liquid has no native support for a log function, so let's get that out of the way first.

There are a couple of different ways to do this, but we're going to pick an easy, brute-force approach: we're going to look up the nearest whole-number answers and then interpolate between them. This will be accurate enough for our purposes and will execute very fast, which is good, since we'll need to do it a _lot_.

Create this file at `_includes/log2.html`:

{% raw %}

```liquid
{% assign value = include.value | plus: 0 %}

{% if value <= 1 %}
  {{ 0.0 }}
{% elsif value <= 2 %}
  {{ value | minus: 1 }}
{% elsif value <= 4 %}
  {{ value | minus: 2 | divided_by: 2.0 | plus: 1 }}
{% elsif value <= 8 %}
  {{ value | minus: 4 | divided_by: 4.0 | plus: 2 }}
{% elsif value <= 16 %}
  {{ value | minus: 8 | divided_by: 8.0 | plus: 3 }}
{% elsif value <= 32 %}
  {{ value | minus: 16 | divided_by: 16.0 | plus: 4 }}
{% elsif value <= 64 %}
  {{ value | minus: 32 | divided_by: 32.0 | plus: 5 }}
{% elsif value <= 128 %}
  {{ value | minus: 64 | divided_by: 64.0 | plus: 6 }}
{% elsif value <= 256 %}
  {{ value | minus: 128 | divided_by: 128.0 | plus: 7 }}
{% elsif value <= 512 %}
  {{ value | minus: 256 | divided_by: 256.0 | plus: 8 }}
{% elsif value <= 1024 %}
  {{ value | minus: 512 | divided_by: 512.0 | plus: 9 }}
{% elsif value <= 2048 %}
  {{ value | minus: 1024 | divided_by: 1024.0 | plus: 10 }}
{% elsif value <= 4096 %}
  {{ value | minus: 2048 | divided_by: 2048.0 | plus: 11 }}
{% else %}
  {{ 12.0 }}
{% endif %}
```

{% endraw %}

This will support meaningful relevance calculations for posts with up to 4,096 common tags and ages up to over 11 years. If you need more than that, you can add more `elsif` blocks.

To use this include, we'll need to capture its output like this:

{% raw %}

```liquid
{% comment %} Assume ageInDays has some numerical value. {% endcomment %}
{% capture logAge %}
  {% include log2.html value=ageInDays %}
{% endcapture %}

{% comment %} Convert the resulting string back to a number. {% endcomment %}
{% assign logAge = logAge | times: 1.0 %}\
```

{% endraw %}

And yes, I am fully aware of how ugly that is. That's Liquid for you! 🤷‍♂️

## Putting It All Together

Here is the replacement for [`page__related.html`](https://github.com/mmistakes/minimal-mistakes/blob/master/_includes/page__related.html) that implements everything described in [Calculating Relevance](#calculating-relevance) above.

See the comments inline for details. Again, my apologies for the awkward layout & syntax... even Shopify's own code formatter can only do so much with Liquid!

Also note the constants at the top of the script. You can adjust these to suit your own needs.

{% raw %}

```liquid
{% comment %} Minimum number of tags a page needs in common with this one to be related. {% endcomment %}
{% assign minCommonTags = 1 %}

{% comment %} Relative weight given to common tags over post recency. {% endcomment %}
{% assign commonTagsWeight = 3 %}

{% comment %} Pages don't have a post date. This is their default age (common tags will boost their relevance). {% endcomment %}
{% assign defaultAge = 32 %}

{% comment %} Maximum number of related pages to display. {% endcomment %}
{% assign maxRelated = 8 %}

{% comment %} Get all pages & collection posts (including blog posts) that are not from the current page's collection unless it is a blog post. {% endcomment %}
{% assign posts = site.pages %}
{% for collection in site.collections %}
  {% assign pageCollection = collection.docs | where: 'url', page.url | size %}
  {% unless pageCollection > 0 and collection.label != 'posts' %}
    {% assign posts = posts | concat: collection.docs %}
  {% endunless %}{% endfor %}

{% comment %} We're going to log-scale & normalize the common tag count, so we need to track its max value. {% endcomment %}
{% assign maxLogCommonTags = 1 %}

{% comment %} Iterate over these comparison pages & capture the relevant ones. {% endcomment %}
{% assign relevantPosts = "" | split: "" %}{% assign now = 'now' | date: '%s' | plus: 0 %}
{% for post in posts %}
  {% comment %} Ignore the current page, hidden pages & redirects. {% endcomment %}
  {% unless post.url == page.url or post.hidden or post.redirect_to %}
    {% comment %} Create a list of tags on the comparison page. {% endcomment %}
  {% assign tags = post.tags | uniq %}

    {% comment %} If some post redirected to this page, include its tags too. {% endcomment %}
    {% assign sourcePost = posts | where: 'redirect_to', post.url | first %}
    {% if sourcePost %}
      {% assign tags = tags | concat: sourcePost.tags | uniq %}
  {% endif %}

    {% comment %} Count the tags the comparison page has in common with this page. {% endcomment %}
    {% assign commonTags = 0 %}
    {% for tag in tags %}
      {% if page.tags contains tag %}
        {% assign commonTags = commonTags | plus: 1 %}
      {% endif %}
    {% endfor %}

    {% comment %} Ignore comparison pages without enough common tags. {% endcomment %}
    {% if commonTags >= minCommonTags %}
      {% comment %} Log-scale the common tag count & update the max value. {% endcomment %}
      {% capture logCommonTags %}
        {% include log2.html value=commonTags %}
    {% endcapture %}
      {% assign logCommonTags = logCommonTags | times: 1.0 %}
      {% if logCommonTags > maxLogCommonTags %}
        {% assign maxLogCommonTags = logCommonTags %}
    {% endif %}

      {% comment %} Create a reference array with the comparison page url & log-scaled common tag count. {% endcomment %}
      {% assign post_data = "" | split: "" %}
      {% assign post_data = post_data | push: post.url | push: logCommonTags %}
      {% assign relevantPosts = relevantPosts | push: post_data %}
    {% endif %}
  {% endunless %}
{% endfor %}

{% comment %} If no comparison pages had enough tags, stop. {% endcomment %}
{% if relevantPosts.size > 0 %}
  {% comment %} Initialize an array for scored posts. {% endcomment %}
  {% assign scoredPosts = "" | split: "" %}

  {% comment %} Iterate over the reference array. {% endcomment %}
  {% for item in relevantPosts %}
    {% comment %} Extract the data. {% endcomment %}
    {% assign post_url = item[0] %}
  {% assign logCommonTags = item[1] | plus: 0.0 %}

    {% comment %} Normalize the log-scaled common tag counts. {% endcomment %}
  {% assign normLogCommonTags = logCommonTags | divided_by: maxLogCommonTags %}

    {% comment %} Go get the post record. {% endcomment %}
  {% assign post = posts | where: "url", post_url | first %}

    {% comment %} Calculate the comparison page's age in days, applying the default when needed. {% endcomment %}
    {% assign postDate = post.date | date: '%s' | plus: 0.0 %}
    {% assign ageInSeconds = now | minus: postDate %}
    {% if ageInSeconds < 60 %}
      {% assign ageInDays = defaultAge %}
    {% else %}
      {% assign ageInDays = ageInSeconds | divided_by: 86400.0 %}
    {% endif %}

    {% comment %} Log-scale the comparison page age. {% endcomment %}
    {% capture logAge %}
        {% include log2.html value=ageInDays %}
    {% endcapture %}
  {% assign logAge = logAge | times: 1.0 %}

    {% comment %} Calculate the relevance score. {% endcomment %}
  {% assign relevance = normLogCommonTags | times: commonTagsWeight | minus: logAge | times: 1000 | plus: 50000 | round %}

    {% comment %} Encode the data in a sortable format & push it to our scored posts array. {% endcomment %}
    {% assign sort_key = relevance | append: "|" | append: post_url | append: "|" | append: normLogCommonTags | append: "|" | append: logAge %}
    {% assign scoredPosts = scoredPosts | push: sort_key %}
{% endfor %}

  {% comment %} Sort the scored posts. {% endcomment %}
  {% assign sorted_relevance_scores = scoredPosts | sort | reverse %}

  {% comment %} Render the related-page HTML wrapper. {% endcomment %}
  <div class='page__related'>
    {% include before-related.html %}
    <h2 class='page__related-title'>
      {{ site.data['ui-text'][site.locale].related_label | default: 'You May Also Enjoy' }}
    </h2>
    <div class='grid__wrapper'>

      {% comment %} Create a counter & iterate over the sorted array. {% endcomment %}
      {% assign maxRelatedCounter = 0 %}
        {% for item in sorted_relevance_scores %}
        {% comment %} Extract the data from the array. {% endcomment %}
        {% assign parts = item | split: "|" %}
        {% assign relevance = parts[0] %}
        {% assign post_url = parts[1] %}
        {% assign normLogCommonTags = parts[2] %}
      {% assign logAge = parts[3] %}

        {% comment %} Go get the post data. {% endcomment %}
      {% assign post = posts | where: "url", post_url | first %}

        {% comment %} Render some gelpful comments. {% endcomment %}
        <!-- post_url: {{ post_url }} -->
        <!-- normLogCommonTags: {{ normLogCommonTags }} -->
        <!-- logAge: {{ logAge }} -->
        <!-- relevance: {{ relevance }} -->

        {% comment %} Render the actual page link. {% endcomment %}
      {% include archive-single.html type="grid" %}

        {% comment %} Update the counter & break if we're done. {% endcomment %}
        {% assign maxRelatedCounter = maxRelatedCounter | plus: 1 %}
        {% if maxRelatedCounter >= maxRelated %}
          {% break %}
        {% endif %}
      {% endfor %}
    </div>
  </div>
{% endif %}
```

{% endraw %}

## Cleaning Up

The updated `page__related.html` widget presented above will give you a super relevant list of related posts & pages at the bottom of any post or page on your Minimal Mistakes site... _assuming that post or page actually displays the widget!_

Whether that widget is actually displayed is controlled by your layout file. For example, this page uses a `single` layout, which is implemented at [`/layouts/single.html`](https://github.com/mmistakes/minimal-mistakes/blob/master/_layouts/single.html).

Right at the bottom of that file, you'll see this block of code:

{% raw %}

```liquid
{% comment %}<!-- only show related on a post page when `related: true` -->{% endcomment %}
{% if page.id and page.related and site.related_posts.size > 0 %}
  {% include page__related.html posts=site.related_posts %}
{% comment %}<!-- otherwise show recent posts if no related when `related: true` -->{% endcomment %}
{% elsif page.id and page.related %}
  {% include page__related.html posts=site.posts %}
{% endif %}
```

{% endraw %}

A couple of issues here:

- `page.id` only has a value on posts, not pages. So including `page.id` in these conditionals turns related-posts display off on pages, even when you've set `related: true` in the front matter.

- The `site.related_posts` array allows you to explicitly set related posts for all pages. If you're doing that, you don't need this widget.

- The old widget accepts an array of posts as an argument: either `site.related_posts` (see above) or `site.posts`, which contains ONLY blog posts, not pages or posts from other collections. The new widget doesn't take any inputs.

So in [`single.html`](https://github.com/mmistakes/minimal-mistakes/blob/master/_layouts/single.html) and any other layout file where you see this code, replace it with this:

{% raw %}

```liquid
  {% comment %}<!-- only show related when `related: true` -->{% endcomment %}
  {% if page.related %}
    {% include page__related.html %}
  {% endif %}
```

{% endraw %}

Easy peasy! Now you can add `related: true` to the front matter of any post or page, and the new widget will display a list of related content at the bottom of the page, **as long as...**

- you've added at least one tag to the post or page, and

- some other post or page has the same tag, and

- if it's a post, it isn't in the same collection as the page you are editing.

As you do this, you are likely to discover that many of your pages and collection posts don't have their `teaser` image or `excerpt` properties set. Add these to improve their look in the related posts widget!
{: .notice--warning}

## Conclusion

The related-posts widget implemented in the code above should satisfy a pretty broad swathe of use cases. Murphy's law says yours is an exception. 🤣

If it is, just have a close look at the code. The math is the math, so any customizations will likely be in the logic that selects which posts and pages to evaluate for relevance.

The code that accomplishes this is ugly (thanks Liquid!) but not unclear. So go nuts, and feel free to drop me a comment if you run into any trouble!
