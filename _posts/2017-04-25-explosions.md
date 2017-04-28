---
layout: post
title: Explosions
---

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/explosion.cljs).

TODO: Reread and explain how the GLSL code works. It involves a quadratic equation for intersection with the ground, making sure the voxels don't fall through it and a height map texture lookup. Also a rotation matrix to make the exploding voxels spin. Plus normals and lighting.

Here is a [screenshot](https://emnh.github.io/rts-blog-screenshots/shots/explosions.jpg):
![screenshot](https://emnh.github.io/rts-blog-screenshots/shots/explosions.jpg)

Watch the [live demo here](https://emnh.github.io/voxel-explosions-demo/#game-test).


