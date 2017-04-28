---
layout: post
title: ClojureScript optimizations
---

In tight render/engine loops Clojure sequences over units turned out to be way too slow, so I had to replace them with plain loops.
I did profiling in Chrome and checked it out. Maybe there were other optimizations but I don't remember.
