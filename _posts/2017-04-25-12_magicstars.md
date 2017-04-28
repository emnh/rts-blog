---
layout: post
title: Magic stars
---

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/magic.cljs).

I thought it would look nice to have magic stars raining down on buildings to indicate they are under construction or some other property, perhaps being under a spell. The GLSL just renders quads facing the camera, billboards as they are called and moves them from a source according to a formula to the location of the voxelized boxes of the unit geometry.

[A screenshot follows:](https://emnh.github.io/rts-blog-screenshots/shots/magic-stars.jpg)
![Magic stars](https://emnh.github.io/rts-blog-screenshots/shots/magic-stars.jpg)
