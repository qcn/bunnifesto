---
layout: default
title: Speculation demo
excerpt_separator: <!--more-->
---

This is an example of a page that speculates.
<!--more-->

It should start speculating when I
hover over <a class="prerender-me" href="{% post_url 2024-11-13-big-aurora-photos %}">this link</a> for a while.

<script type="speculationrules">
    {
        "prerender": [
            {
                "where": { "selector_matches": ".prerender-me" },
                "eagerness": "moderate"
            }
        ]
    }
</script>
