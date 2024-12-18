---
layout: post
title: Speculation demo
excerpt_separator: <!--more-->
---

This is an example of a page that speculates.
<!--more-->

These are some links to photos of the sky on this blog. They are configured to prerender with
the indicated [eagerness]:
* [May 2024 aurora]({% link _talks/speculation-rules/big-may-2024-aurora-photos.md %}) [moderate]
* [August 2024 aurora]({% link _talks/speculation-rules/august-2024-aurora-photo.md %}) [moderate]
* [October 2024 aurora]({% link _talks/speculation-rules/october-2024-aurora-photo.md %}) [moderate]
* <a class="prerender-me" href="{% link _talks/speculation-rules/milky-way-photo.md %}">Milky Way</a> [eager]

Here are some other links that are configured to prefetch with `eagerness:
immediate`:
* <a class="prefetch-me" href="{% post_url 2024-05-16-site-in-progress
  %}">Another page on this site (same-site prefetch)</a>
* <a class="prefetch-me" href="https://www.abc.net.au/news">ABC News website
  (cross-site prefetch)</a>

<script type="speculationrules">
    {
        "prerender": [
            {
                "where": { "selector_matches": ".prerender-me" },
                "eagerness": "eager"
            },
            {
                "where": { "href_matches": "/*aurora*" },
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
