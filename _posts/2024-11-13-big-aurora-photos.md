---
layout: post
title: Big aurora photos!
excerpt_separator: <!--more-->
---

Here are some large photos of an aurora.
<!--more-->

Time from activation to page definitely loaded: <span id="load-time"></span>

![May 11, 2024 aurora](/assets/images/aurora/20240511-aurora-01.jpg)

![May 11, 2024 aurora](/assets/images/aurora/20240511-aurora-02.jpg)

![May 11, 2024 aurora](/assets/images/aurora/20240511-aurora-03.jpg)

Here is a bonus photo of the Milky Way:

![Milky Way](/assets/images/astro/20230514-milky-way.jpg)

<script type="text/javascript">
    if (document.prerendering) {
        document.addEventListener(
            "prerenderingchange",
            () => requestAnimationFrame(renderLoadTime),
            { once: true },
        );
    }
    else {
        requestAnimationFrame(renderLoadTime);
    }

    // This isn't exactly the "load time", but it's as close to it as I can get
    // for the purposes of this demo, because it's hard to work out when a page
    // has finished painting!
    // Print the time since this page was activated, which is 0 if not
    // prerendered and the timestamp of activation if it was prerendered/in the
    // process of prerendering, keep going until the load event has been fired.
    function renderLoadTime() {
        var navPerformanceEntry = window.performance.getEntriesByType("navigation")[0];
        var now = window.performance.now();
        var loadTime = now - navPerformanceEntry.activationStart;
        document.getElementById("load-time").textContent = (loadTime/1000).toFixed(3);
        if (!!navPerformanceEntry.loadEventEnded && navPerformanceEntry.loadEventEnded < now) {
            requestAnimationFrame(() => renderLoadTime());
        }
    }
</script>

