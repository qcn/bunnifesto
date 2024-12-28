---
layout: post
title: Speculative Loading and the Speculation Rules API
excerpt_separator: <!--more-->
---

*Happy holidays! This article was written from a talk I gave at [Web
Directions](https://webdirections.org/) Developer Summit
2024, and originally published as [Day 18 of the 2024 Planet Performance Web
Performance Advent
Calendar](https://calendar.perfplanet.com/2024/speculative-loading-and-the-speculation-rules-api/). Thanks to John Allsopp for the idea, Stoyan
Stefanov for the original platform, and Barry Pollard for reviewing the article
before publication!*

[Speculative loading](https://developer.mozilla.org/en-US/docs/Web/Performance/Speculative_loading), or navigation speculation, is the concept of predicting (*speculating* about) which page a user might visit next and doing some or all of the work to load that page before they visit it. This way, if the user does end up navigating to the page, it appears to load much faster. Browsers have offered several types of resource hints to enable speculation for many years, but many of these interfaces have been clunky and inflexible to use. Google has recently introduced the Speculation Rules API in Chromium, designed to make it easier to prefetch and prerender documents for future navigations.

<!--more-->

When you navigate to a URL, the flow for the browser looks something like:

* do a DNS lookup on the domain name
* execute the TCP handshake to open a connection to the server
* execute the TLS handshake to agree on how to secure the connection (at least, I hope it's doing this!)
* request the document from the server
* receive document from server
* parse document, request and receive subresources, execute scripts, etc
* paint final result to your screen

There are many stages in this workflow, and existing resource hints allow us to carry out several of them in advance. However, these often have limitations which should be understood to avoid wasted work. For example:

* [**dns-prefetch**](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/dns-prefetch) does the DNS lookup in advance
* [**preconnect**](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/preconnect) does the DNS lookup, TCP handshake, and TLS handshake, so the connection is ready to go when a resource is requested; however, browsers close an unused connection after 10 seconds, so you shouldn't preconnect unless you're going to use the connection soon.
* [**prefetch**](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/prefetch) fetches resources so they're already cached and don't need to be downloaded on navigation. This relies on the browser's HTTP cache, so it's subject to being blocked by Cache-Control headers, and can't be used for cross-site speculation due to [cache partitioning](https://developer.chrome.com/blog/http-cache-partitioning).

These speculations also require the developer to explicitly identify the resource/URL that should be speculated on, and trigger the speculation itself. For example, this might be based on user behaviour: you could insert a new `<link rel="prefetch">` tag to prefetch the target when a link is hovered over or clicked on. Another approach might be to use your site's navigation metrics to determine likely navigation flows based on user activity, and speculate on the most likely next destinations.

Google's [Speculation Rules API](https://wicg.github.io/nav-speculation/speculation-rules.html), which is available in Chromium-based browsers since Chrome 109, offers a more ergonomic and full-featured approach to prefetch and prerender. As of December 2024, there is no support yet in Firefox (see the [tracking issue](https://github.com/mozilla/standards-positions/issues/620) for their position on the feature, which they recently changed to "positive"/"interested in implementing" for prefetch and "neutral" for prerender) or Safari ([tracking issue here](https://github.com/WebKit/standards-positions/issues/54)). However, as it's a progressive API, you can use it and browsers that don't support it will simply ignore it. You can use feature-detection to fall back to supported forms of speculation if you wish.

## What's different about the Speculation Rules API?

Prerender is the big deal when it comes to Speculation Rules. Chrome itself now prerenders some pages when you use the browser's address bar to navigate or search, and when you navigate using the bookmark bar. The Speculation Rules API allows developers to trigger the same kind of prerender to load their own pages.

While there's a legacy prerender interface [`<link rel="prerender">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/prerender) (which is, like Speculation Rules, only supported in Chromium) it doesn't trigger a full prerender -- the feature was originally implemented as a full prerender but as this was too resource-heavy and complex to support, it was replaced with [NoState Prefetch](https://developer.chrome.com/blog/nostate-prefetch). This type of prefetch will fetch the specified resource and a limited set of its subresources, populating the HTTP cache as for `<link rel="prefetch">`, but doesn't render the page or execute any scripts.

Speculation Rules prerender, on the other hand, *is* a full prerender. When a Speculation Rules prerender is triggered, Chrome renders the target page in an invisible tab, using a per-document, in-memory cache. When the user subsequently navigates to that prerendered page, the browser makes that invisible tab active; the resulting user experience is that the page loaded (almost) instantly, because it was already rendered and everything was in memory. This is beneficial even if the user navigates before the page finishes prerendering: the browser still activates the invisible tab immediately and continues the rendering process, and since the page was already partially loaded, the user doesn't have to wait as long for it to load from scratch.

Because this method actually renders the page and executes most JavaScript, it's also able to load subresources that are loaded via JavaScript, unlike `<link rel="prerender">`.

Speculation Rules prefetch uses the same per-document in-memory cache as prerender to store prefetched documents. Since it doesn't rely on the HTTP cache, it doesn't get blocked by Cache-Control headers, but it can still populate the HTTP cache given suitable response headers. However, as it's a document-based prefetch and doesn't parse the document, it can't prefetch subresources -- if you want to do that, you'll still need to use one of the existing approaches where you can specify the subresources explicitly.

Another tradeoff of the Speculation Rules API being designed for navigation between documents is that it works only for multi-page apps. Single page apps don't do real page navigations that the browser manages, even if they're using a router that makes it look like the page URL is changing -- they only fetch new data to render within the current document. So you could prerender the single page app *itself* from a previous page, and that could help with the initial load time of the app, but you can't use Speculation Rules for navigations *within* it.

As previously mentioned, legacy approaches to speculation are fairly inflexible in how resources to be speculated on are specified -- the developer needs to specify the exact resource to be fetched, and determine when to trigger speculation. The Speculation Rules syntax is more flexible, and it allows the browser to do a lot of the work in deciding what to speculate on and when.

Under the Speculation Rules API, you use JSON to define *rules* that tell the browser what to speculatively load. You can still specify exact URLs, but you can also define rules about how to *match* links in the current page: the browser will scan over all the anchor and area tags in the document to look for links, and find the ones that match the rules. You can pattern-match URLs and match by CSS selector, and you can combine and negate conditions. This is much nicer than having to create `<link>` elements for each URL explicitly.

Additionally, you can set an eagerness level for each rule, which hints how important this speculation is and lets the browser take care of it. In order of importance, these levels are:

* `immediate` -- the browser should trigger this speculation as soon as possible
* `eager` -- at present, Chrome treats this the same way as immediate
* `moderate` -- currently triggered after hovering the link for 200ms (or on `pointerdown` if that's earlier)
* `conservative` -- to minimise resource wastage, the browser should only start the speculation when the user activates the link (on touch or pointer down, a few milliseconds ahead of the actual click being registered)

While Chrome currently implements these eagerness levels as described above, future browser developers could choose their own heuristics for determining an increased likelihood of navigating to the link. You should consider the characteristics of your site when you're choosing how eager the speculation should be -- if your pages are just static HTML, speculating on that is going to create a lot less overhead than if you have really heavy pages with a lot of Javascript, where you might want to be more conservative because the impact of that potential wasted work is higher.

Note that Chrome will also consider resource constraints and user settings in determining how to prioritise speculation. In addition to users being able to disable speculation completely, there are [several reasons](https://source.chromium.org/chromium/chromium/src/+/main:content/public/browser/preloading.h?q=PreloadingEligibility&ss=chromium) why Chrome might disable or deprioritise speculation, including (but certainly not limited to) the computer being in battery saver mode or low on memory. Note that not all the reason codes are implemented in Chrome (for example, "low bandwidth" isn't implemented); additionally, some (such as service workers) apply only to prefetch, and others (cookies) apply only to cross-origin prefetches.

There are also [limits](https://developer.chrome.com/docs/web-platform/prerender-pages#chrome-limits) to how many speculations will be done at a time. For example, the browser will queue at most two moderate or conservative speculations. If a third non-eager speculation is triggered, the first one will be cancelled, because the user is more likely to follow the link they're currently interacting with.

## How to use the Speculation Rules API

Rules can be inserted directly into the page using a [`<script type="speculationrules">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/speculationrules) element, eg:

```javascript
<script type="speculationrules">
    {
        "prerender": [
            {
                "where": { "selector_matches": ".prerender-me" },
                "eagerness": "eager"
            },
            {
                "where": { "href_matches": "/*bunnypics*" },
                "eagerness": "moderate"
            }
        ],
        "prefetch": [
            {
                "where": { "selector_matches": ".prefetch-me" },
                "eagerness": "immediate"
            }
        ]
    }
</script>
```

or defined in one or more files and specified using a [Speculation-Rules response header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Speculation-Rules), which allows use of the API without having to change the document itself:

```
Speculation-Rules: "/rules/prefetch.json","/rules/prerender.json"
```

## Limitations and considerations

As the Speculation Rules API is specifically designed for navigation, it can sometimes be used to speculate on [cross-site](https://developer.mozilla.org/en-US/docs/Glossary/Site) or [cross-origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin) navigation. Specifically:

* The **site** is a boundary for **privacy**. Same-site *prefetches*, whether same- or cross-origin, are allowed, because prefetch doesn't execute scripts and the speculation doesn't cross the privacy boundary. However, *cross-site* prefetches only work if the browser doesn't already have cookies set for the destination site. This prevents sites from tracking user activity through prefetches, using existing cookies. In future, Chromium may allow cross-site prefetches via opt-in from the destination server.
* The **origin** is a boundary for **security**. *Prerender* executes scripts which may be a security risk, so while same-origin prerender is permitted, cross-origin/same-site prerender is only allowed if the destination server opts in using the [Supports-Loading-Mode response header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Supports-Loading-Mode). Cross-site prerender is not allowed.

However, even with the built-in limitations, it's important to note that there are still privacy considerations when using speculation! Prefetching a page intrinsically provides the destination server with information about who the user is and what page they're currently browsing -- in fact, the act of prefetching in itself could reveal information about their setup. For example, since speculation is blocked in energy saver mode, the act of speculating implies the user isn't using that mode. Similarly, if the speculation rules are based on analyzing user activity and predicting where people with similar usage patterns go next, making a prefetch request could leak information about what kind of pages the user has visited. It's important to consider the privacy tradeoffs when choosing to use speculation -- the API alone won't protect you from everything. For reasons like these, speculation is often disabled in the browser by privacy-focused extensions like [uBlock Origin](https://github.com/gorhill/uBlock/wiki/Dashboard:-Settings#disable-prefetching).

You should also keep in mind that even though prefetching and prerendering can make *navigation* faster for your users, it's not a substitute for optimising your download sizes. That data is going to be transferred whether you do it before or after navigation, so it's still subject to all the normal constraints, like slow internet connections or data transfer limits -- and speaking as a platform engineer, you're still paying for the bandwidth!

This makes speculative loading a bit of a gamble. If it works out well, it can have great benefits and your users will love you for making their experience so fast! But if you load the wrong things, it'll cost you and your users, in wasted data transfer, resource usage, and potentially extreme confusion.

The obvious case is if the user doesn't end up visiting that page next. Everything else you prefetch or prerender gets discarded from the in-memory cache if you navigate to a different page, so if you prerender two possible next destinations from the current page, at least one of those is going to be thrown out on navigation, maybe *both*. It doesn't matter if there would have been a subsequent navigation to the other page -- the cached prerender is already gone! If your prerender stored anything in the HTTP cache, that will still be there, but even with this small benefit you still probably want to avoid a wasted prerender because it's doing so much background work. You might decide you're okay with this scenario for a prefetch, because it's much lighter on resources.

However, the page might also change before it's loaded! Consider the case of prerendering a thread on a discussion forum, where the comments depend on server state. If another user posts a new comment after your browser prerenders the thread and then you navigate to the prerendered page, the comments are out of date -- that isn't so bad. But if you're a moderator on the forum and you just deleted a comment, then you navigate to the prerendered thread and it looks like it's still there, that's confusing.

That also applies for state that's changed by the user. Say you visit a site that defaults to dark mode, and you switch to light mode. Your preference is saved locally in your browser, without reloading the page. Then you navigate to the next page, but it was prerendered before you changed to light mode, so it shows up in dark mode -- that's confusing too!

In these situations, it's best for pages to know how to refresh themselves if the state might change. However, you won't always have control over the pages you're prerendering or the server they come from, so you need to be aware of these possibilities.

It gets worse, though -- these examples demonstrated the user seeing an inconsistent state, which was inconvenient but mostly confusing to the user. But another pattern where it's not safe to prerender or prefetch is where making a request for a page has side effects -- it initiates some action or has some effect that you don't want to trigger until you actually navigate there. For example, requesting the page could log the user out, change their preferred language, or use up free articles on a paywalled site. If you have control over the server those pages are coming from, you can detect that the request is because of a prefetch or prerender and defer those actions until the navigation actually happens.

Similarly, there are [a number of web and browser APIs and features](https://developer.mozilla.org/en-US/docs/Web/API/Speculation_Rules_API#platform_features_deferred_or_restricted_during_prerender) that are deferred or restricted when prerendering by default, because they trigger behaviours that shouldn't activate until the page is activated. For example, you don't want video or audio to start playing for a page you haven't navigated to yet!

## Debugging speculation rules

Fortunately for developers, Chrome has implemented new developer tools that help to see how speculation is working, but are particularly handy to figure out why it's not working. They provide [a useful guide for debugging speculation rules](https://developer.chrome.com/docs/devtools/application/debugging-speculation-rules) in their developer documentation.

## See it in action

I used [this speculation rules demo]({% link _talks/speculation-rules/demo.md %}) for the Web Directions talk that this article was based on. I'll link the video here for more context when it's available, but feel free to contact me if you have any questions!

### Further reading:

* [Speculation Rules
  API](https://developer.mozilla.org/en-US/docs/Web/API/Speculation_Rules_API) at MDN Web Docs
* [Prerender with the Speculation Rules
  API](https://developer.chrome.com/docs/web-platform/prerender-pages) at Chrome for Developers
* [Speculation Rules draft
  specification](https://wicg.github.io/nav-speculation/speculation-rules.html) at WICG
* [Prerendering Revamped draft
  specification](https://wicg.github.io/nav-speculation/prerendering.html) at WICG
