---
layout: post
title: "Why I still prefer CSS over heavy visual frameworks"
excerpt: "Modern web tools prioritize complex utility runtimes, but bare CSS continues to offer unmatched speed and longevity. A retrospective look at styling for performance and system-level simplicity."
date: 2026-02-12 09:00:00 +0000
---

Every year, a new set of CSS-in-JS frameworks or massive utility classes sweeps the web-development landscape. They promise to solve modular scoping, eliminate naming anxiety, and optimize final bundles automatically. However, these systems come with severe architectural tradeoffs.

By introducing additional compile-time pipelines and heavy JavaScript runtimes, we distance ourselves from native browser capabilities. Plain CSS has evolved significantly. With modern custom properties, native nesting, and container queries, vanilla stylesheets are now incredibly robust and maintainable.

## The weight of modern compilation

When we rely on external systems to write simple styling rules, we enter a state of dependency. If the compiler changes its specifications, or the library maintainers drop backward compatibility, entire legacy codebases break silently. Vanilla CSS, on the other hand, is guaranteed to render identically for decades.

For content-focused sites—like this personal technical blog—the fastest stylesheet is the one the browser natively parses. By avoiding runtime style computation, we achieve perfect layout performance and reduce layout shifts to absolute zero.
