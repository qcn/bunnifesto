---
layout: post
title: Speculation demo
excerpt_separator: <!--more-->
---

This is an example of a page that speculates.
<!--more-->

These are some links to photos of the sky:
* [May 2024 aurora]({% post_url 2024-11-13-big-aurora-photos %})
* [August 2024 aurora]({% post_url 2024-11-16-august-aurora %})
* [October 2024 aurora]({% post_url 2024-11-16-october-aurora %})
* <a class="prerender-me" href="{% post_url 2024-11-16-milky-way-photo %}">Milky Way</a>

Here are some other links:
* <a class="prefetch-me" href="{% post_url 2024-05-16-site-in-progress
  %}">Another page on this site</a>
* <a class="prefetch-me" href="https://www.abc.net.au/news">A
  page elsewhere</a>

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
