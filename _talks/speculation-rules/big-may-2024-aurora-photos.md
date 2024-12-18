---
layout: post
title: May 11, 2024 Aurora Australis
excerpt_separator: <!--more-->
---

Here are some large photos of an aurora for loading demonstration purposes.
<!--more-->

![May 11, 2024 aurora](/assets/images/aurora/20240511-aurora-01.jpg)

![May 11, 2024 aurora](/assets/images/aurora/20240511-aurora-02.jpg)

![May 11, 2024 aurora](/assets/images/aurora/20240511-aurora-03.jpg)

<script type="text/javascript">
const observer = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1]; // Use the latest LCP candidate
  console.log("Activation to LCP render time (seconds):", (lastEntry.renderTime - window.performance.getEntriesByType('navigation')[0].activationStart) / 1000);
  console.log(lastEntry);
});
observer.observe({ type: "largest-contentful-paint", buffered: true });
</script>
