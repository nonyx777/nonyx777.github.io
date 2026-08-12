---
layout: post
title: "Understanding memory leaks in serverless runtimes"
excerpt: "Serverless functions are short-lived, but global scope leakage can accumulate latency penalties across cold starts. Exploring node-inspect profiles and standard visual leak patterns."
date: 2026-01-24 09:00:00 +0000
---

Serverless functions are short-lived, but global scope leakage can accumulate latency penalties across cold starts. This post explores node-inspect profiles and standard visual leak patterns.
