---
layout: post
title: Big aurora photos!
---

Seconds to load: <span id="load-time"></span>

Here are some large photos of an aurora.

![May 11, 2024 aurora](/assets/images/aurora/20240511-aurora-01.jpg)

![May 11, 2024 aurora](/assets/images/aurora/20240511-aurora-02.jpg)

![May 11, 2024 aurora](/assets/images/aurora/20240511-aurora-03.jpg)

Here is a bonus photo of the Milky Way:

![Milky Way](/assets/images/astro/20230514-milky-way.jpg)

<script type="text/javascript">
    requestAnimationFrame(renderLoadTime);
    function renderLoadTime() {
        var navPerformanceEntry = window.performance.getEntriesByType("navigation")[0];
        var loadTime = navPerformanceEntry.duration || window.performance.now();
        document.getElementById("load-time").textContent = (loadTime/1000).toFixed(3);
        if (navPerformanceEntry.duration === 0) {
            requestAnimationFrame(() => renderLoadTime());
        }
    }
</script>

