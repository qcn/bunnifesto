---
layout: post
title: Speculation demo
excerpt_separator: <!--more-->
---

This is an example of a page that speculates.
<!--more-->

These are some links to photos of the sky on this blog. They are configured to prerender with
the indicated [eagerness]:
* [May 2024 aurora]({% post_url 2024-11-13-big-may-aurora-photos %}) [moderate]
* [August 2024 aurora]({% post_url 2024-11-16-02-august-aurora %}) [moderate]
* [October 2024 aurora]({% post_url 2024-11-16-03-october-aurora %}) [moderate]
* <a class="prerender-me" href="{% post_url 2024-11-16-01-milky-way-photo %}">Milky Way</a> [eager]

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
