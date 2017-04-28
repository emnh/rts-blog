---
layout: post
title: Voxelization of 3D geometries
---
First off, [here is a link to the source code](https://github.com/emnh/rts/blob/master/src.client/game/client/voxelize.cljs). Voxelization means to represent each part of a 3D geometry as a small box of the same size, rather than triangles of varying size. I used the approach described [here](http://drububu.com/miscellaneous/voxelizer/index.html). Well, it seems the author of that page has removed the description of the algorithm and just left the online voxelizer. Anyway, the algorithm is not too complicated: it runs entirely on the CPU (no GPU) and involves recursively subdividing each triangle in the 3D geometry until a constant threshold is hit, then check which box the triangle lies in and mark it as active / on. In addition, I did a flood fill to mark all interior boxes as on as well, since I wanted to explode the voxels and thus needed the inside filled. The algorithm is quite simple and slow, taking up to an hour to voxelize a simple 3D model, but I don't do it realtime, I just run it offline as a script with node.js over-night and save the voxels for the game to load, so it doesn't matter that much. If you want to do realtime voxelization you should look [here](https://developer.nvidia.com/content/basics-gpu-voxelization) instead.

Later I decided to add UV coordinates for texture mapping to each box relative to the whole 3D model so that I can texture each box as close to the original geometry as possible. I proceeded to use the same recursive subdivision of each triangle T, then for each corner V of the small triangle and for each corner C of the box which V resides in, I update the UV of the box corner C (really, the UV of the box which has this corner as top left corner) if the closest point P on the big triangle T containing V is the current closest to C, and also save T as the closest triangle to C. I update the UV to the interpolation of UVs from the corners of the subdivided triangle T from the original 3D model using barycentric coordinates and P. As an optimization, for each box corner I also maintain a set of seen triangles and skip any triangle that has been seen before, since we already know what the closest point of that triangle to the box corner is. It's a naive algorithm in that it's the first thing that popped into my head, even if I had a bit trouble understanding the code that I wrote before now, and I am sure there are better approaches, but at least it gave results resembling the texturing of the original triangle 3D model.

[Before voxelization screenshot:](https://emnh.github.io/rts-blog-screenshots/shots/before-voxelization.jpg)
![Before voxelization](https://emnh.github.io/rts-blog-screenshots/shots/before-voxelization.jpg)
[After voxelization screenshot:](https://emnh.github.io/rts-blog-screenshots/shots/after-voxelization.jpg)
![After voxelization](https://emnh.github.io/rts-blog-screenshots/shots/after-voxelization.jpg)

TODO: Add some illustrative schematics.
TODO: Describe how to edit and run the voxelization script using node.js from the commandline.


