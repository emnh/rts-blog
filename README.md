# RTS Blog

I am creating a [real-time strategy (RTS) game in WebGL and ClojureScript](https://github.com/emnh/rts). This is my blog about the process. I made the mistake of not documenting my development initially, but I will start now (25.04.2017) and try to look back on the code I've already written. If you want to comment on this blog [open an issue](https://github.com/emnh/rts-blog/issues) per topic or find an existing one on the GitHub repository. I might attach issue comments to the blog later using [this process](http://donw.io/post/github-comments/).

The following [screenshot](https://emnh.github.io/rts-blog-screenshots/shots/game-test.jpg) will be updated as the game progresses (historical images will be in the blog posts about each feature):
![screenshot of rts](https://emnh.github.io/rts-blog-screenshots/shots/game-test.jpg)

## Index of blog posts / topics
 - [History](#history)
 - [Game plan](#gameplan)
 - [ClojureScript component architecture](#components)
 - [ClojureScript React library: Rum](#webui)
 - [Single-page application](#spa)
 - Resource downloading and progress manager
 - [Bootstrapping ClojureScript in a web worker](#worker-bootstrap)
 - Engine logic web worker separation and message passing
 - [ClojureScript optimizations](#optimizations)
 - [Camera control](#camera)
 - [Terrain](#terrain)
 - [Fast heightfield lookup](#height-lookups)
 - [Voxelization of 3D geometries](#voxelization)
 - [Unit explosions using voxelized representation](#explosions)
 - [Magic stars](#magicstars)
 - [Health bars](#healthbars)
 - [Marquee selection](#marquee)
 - [Water](#water)
 - [Minimap](#minimap)
 - [Attack vectors using MathBox](#mathbox)

## Blog posts / topics

### <a name="history">History</a>

Initially the game was written in JavaScript with Babel, the ES2015 and beyond to ES5 compiler. I switched (more or less complete rewrite) to ClojureScript because the mutable state everywhere was becoming a mess, for hot code reloading and because Babel compilation times were too high (I think on the order of 30 seconds, but I don't remember so well). I still haven't implemented feature parity with the original version, but I've been focusing on new features like fancy terrain and water instead and notably Starcraft 2 3D model support has been dropped. I suppose the Starcraft 2 on WebGL headline was sparking some of the initial interest, getting a [Hacker News submission](https://news.ycombinator.com/item?id=10205347) on the front page and around 13k views, but support for non-free models is by far not the primary goal of the project. Developing a strong and independent engine is.

Here is a [screenshot](https://emnh.github.io/rts-blog-screenshots/shots/rts-free.jpg) of the version with free models:
![screenshot of rts-free](https://emnh.github.io/rts-blog-screenshots/shots/rts-free.jpg)

[Here is the live version](http://emh.lart.no/publish/rts-free.git/).

The version which downloaded SC2 models from another host is not working anymore since the host is not serving the files anymore, so all you can do is watch YouTube videos of it (or if you are brave check out the [legacy branch of rts](https://github.com/emnh/rts/tree/legacy) and try to get it to work):

 - [StarCraft 2 on WebGL](https://www.youtube.com/watch?v=PoPNrz2LUG0)
 - [Starcraft 2 on WebGL with colorful UV mapping](https://www.youtube.com/watch?v=EvhUteDp3o8)
 - [Banelings, banelings, banelings](https://www.youtube.com/watch?v=aqKsVelmeeI)

### <a name="gameplan">Game plan</a>

My RTS will at least contain the major engine components that an RTS needs. We'll see what it ends up as. It might become just an engine with a simple tower defense as a demo or something totally different, depending on technical hurdles and wild ideas coming up during development experiments. I am not sure whether I will support multiplayer yet, or whether I will make a series of single player tutorial steps where you write JavaScript or ClojureScript to control the AI of the game to overcome various scenarios (test cases), like CodingGame, but with fancier graphics and more complex rules similar to a commercial RTS. Currently I am leaning towards the latter, because it allows more freedom about simulation speed and no synchronization issues, but at the same time I want the game to be deterministic with a replay function, so I can't generate too much simulation data. It will be hard to tack on multiplayer support later, so I need to make the decision before I start implementing the engine logic beyond the graphics. Currently I've been focusing on the graphics. I defer naming my game until I know what it will be about. I detest having to repeat myself in strategy games building the base, so I want it to be able to be automated as well as avoid any stressful micro with AI scripts, but I still want the game to be able to have human interaction with scripts as executable in-game tasks. The game should be designed such that the best human strategy should win over any AI macro/micro helper scripts.

See also [Design.md](https://github.com/emnh/rts/blob/master/Design.md) for more random thoughts.

### <a name="components">ClojureScript component architecture</a>

I chose [Stuart Sierra's component framework](https://github.com/stuartsierra/component) for ClojureScript to implement the game components. This accomplishes basically the same as a dependency injection framework: all the dependencies, the wiring together of components, are specified at the top level and allows for more easy reconfiguration. A component is a record with a protocol (interface) for start and stop methods. Whenever you change a file, figwheel reloads the scripts for the page and using the component library I  restart by calling stop on each component in the system, in reverse dependency order, and then start on each one in forward order. In practice, for most heavy components, like the resource loader for textures and geometries, I do nothing in the stop handler and in the start handler I only load the resources if they haven't been loaded before. I only do a real stop and start for light components, and for the component which I am currently working on. I should have been more diligent about implementing proper stop handlers and a system for changing which components get reloaded and not, as I currently have to do a full page reload for most changes, so I'm losing hot reloading. I just got lazy about it. I need to do some refactoring to address the issue, but page reloads are not much slower currently than a figwheel reload. Refactoring for hot reloading will pay off if I am in the middle of a game and want to change something however, but I want to be able to serialize game state anyway, so perhaps saving to localStorage and page reload will do.

I made a helper macro for creating components called [defcom](https://github.com/emnh/rts/blob/master/src.shared/game/shared/macros.clj) that defines a component record along with its dependencies, since I will know them up front. It also adds :start-count, :stop-count and :started properties to each component. Most functions still take a component as input, which is basically a dictionary (in ClojureScript record form). So functions take dictionaries and return dictionaries, which makes it easy to add arguments.

All state is kept in one global dictionary, but again, I haven't been diligent about separating updatable atoms and JS objects from serializable state. I need to think more about this and refactor to address this as well, kind of like a virtual DOM that separates input state from UI objects for three.js, recreating and deleting UI objects as units are updated for example. I also look for inspiration from the [redux project](https://github.com/reactjs/redux), to be able to replace local component atoms with a grand central dispatch which updates the global nested state dictionary.

At one point I investigated a lot of alternatives to Stuart's component (list taken from [danielsz system](https://github.com/danielsz/system)):
- [modular](https://github.com/juxt/modular)
- [leaven](https://github.com/palletops/leaven) and [bakery](https://github.com/palletops/bakery])
- [yoyo](https://github.com/james-henderson/yoyo)
- [hara.component](http://docs.caudate.me/hara/#haracomponent)
- [mount](https://github.com/tolitius/mount)

They all have advantages and disadvantages, but in the end I decided not to switch (yet). I've been looking hard at [plumatic plumbing graph](https://github.com/plumatic/plumbing) to wire together dependencies lately, and I might switch to that, but I haven't felt like refactoring yet.

### <a name="webui">ClojureScript React library: Rum</a>

[Source code of example usage is here](https://github.com/emnh/rts/blob/master/src.client/game/client/page_game_test.cljs). The settings on the right of the screenshots, like "Water Depth Effect", are all Rum.

I chose to use a ClojureScript React library called [Rum](https://github.com/tonsky/rum). I don't remember why I chose this over [Om](https://github.com/omcljs/om) or [Reagent](https://github.com/reagent-project/reagent). I think it might have been this point from the Rum readme: "No enforced state model: Unlike Om, Reagent or Quiescent, Rum does not dictate where to keep your state. Instead, it works well with any storage: persistent data structures, atoms, DataScript, JavaScript objects, localStorage or any custom solution you can think of." Anyway the point of using any such React library over plain HTML for me is that hot reloading comes for free. This does not apply to the three.js canvas though.

### <a name="spa">Single-page application</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/routing.cljs) and 
[here](https://github.com/emnh/rts/blob/master/src.client/game/client/core.cljs).

I used the [goog.History library](https://google.github.io/closure-library/api/goog.History.html) and wrote a custom solution for loading pages. Each page is a component, and also the game page component has a subsystem which contains the game components. I have a div for each page and mount Rum on it when the component starts. The single-page app code is a bit hairy, especially some hacks to differentiate switching a page and figwheel reloads and avoiding infinite reload loops, and since only one page should be loaded at a time starting the system is different from the standard Stuart Sierra component library call to start the system. It seems to work fine now however, so I am leaving a potential cleanup for later.

[Not much of a screenshot follows:](https://emnh.github.io/rts-blog-screenshots/shots/lobby.jpg)
![Lobby](https://emnh.github.io/rts-blog-screenshots/shots/lobby.jpg)

### <a name="worker-bootstrap">Bootstrapping ClojureScript in a web worker</a>

It is quite slow, taking about 5 seconds to load the worker, and there is no figwheel hot reload support.
Here is how to bootstrap ClojureScript in a Web Worker:

```javascript
console.log("from worker");
console.time("worker-load")

CLOSURE_BASE_PATH = "../goog/"
/**
 * Imports a script using the Web Worker importScript API.
 *
 * @param {string} src The script source.
 * @return {boolean} True if the script was imported, false otherwise.
 */
this.CLOSURE_IMPORT_SCRIPT = (function(global) {
  return function(src) {
    global['importScripts'](src);
    return true;
  };
})(this);

BASE_PATH = "../";
importScripts(CLOSURE_BASE_PATH + "base.js");
importScripts("../game.js");
importScripts(
    BASE_PATH + "jscache/simplex-noise.js",
    BASE_PATH + "jscache/three.js",
    BASE_PATH + "bundle-deps-worker.js");
goog.require('game.worker.core');
console.timeEnd("worker-load")
```

### <a name="optimizations">ClojureScript optimizations</a>

In tight render/engine loops Clojure sequences over units turned out to be way too slow, so I had to replace them with plain loops.
I did profiling in Chrome and checked it out. Maybe there were other optimizations but I don't remember.

### <a name="camera">Camera control</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/controls.cljs).

I implemented typical RTS controls. Pan left/right/up/down, zoom in/out and also arcball rotation. Worth noting is that you should only move the camera for each requestAnimationFrame to make smoother animations, not on the keypress events themselves. The more interesting implementation was arcball rotation, which I think I got [from here](https://raw.githubusercontent.com/grey-eminence/3DIT/master/js/controls/MapControls.js):

```clojure
(defn arc-ball-rotation-left-right
  [state sign]
  (let
    [camera (:camera state)
     focus (scene/get-camera-focus camera 0 0)
     axis (-> camera .-position .clone)
     _ (-> axis (.sub focus))
     _ (-> axis .-y (set! 0))
     _ (-> axis .normalize)
     old (-> camera .-position .clone)
     config (:config state)
     rotate-speed (get-in config [:controls :rotate-speed])
     rotate-speed (* sign rotate-speed)
     rotate-speed (* rotate-speed (get-elapsed state))]

    (-> camera .-position (.applyAxisAngle axis rotate-speed))
    (-> camera .-position .-y (set! (-> old .-y)))
    (-> camera (.lookAt focus))))


(defn arc-ball-rotation-up-down
  [state sign]
  (let
    [camera (:camera state)
     focus (scene/get-camera-focus camera 0 0)
     axis (-> camera .-position .clone)
     _ (-> axis (.sub focus))
     offset axis
     config (:config state)
     rotate-speed (get-in config [:controls :rotate-speed])
     rotate-speed (* sign rotate-speed)
     rotate-speed (* rotate-speed (get-elapsed state))
     theta (atan2 (-> offset .-x) (-> offset .-z))
     xzlen (sqrt (+ (square (-> offset .-x)) (square (-> offset .-z))))
     min-polar-angle 0.1
     max-polar-angle (- (/ pi 2) (/ pi 16))
     phi (atan2 xzlen (-> offset .-y))
     phi (+ phi rotate-speed)
     phi (min max-polar-angle phi)
     phi (max min-polar-angle phi)
     radius (-> offset .length)
     x (* radius (sin phi) (sin theta))
     y (* radius (cos phi))
     z (* radius (sin phi) (cos theta))
     _ (-> offset (.set x y z))]

    (-> camera .-position (.copy focus))
    (-> camera .-position (.add offset))
    (-> camera (.lookAt focus))))
```

### <a name="terrain">Terrain</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/ground_local.cljs)
 and [here](https://github.com/emnh/rts/blob/master/src.client/game/client/ground_fancy.cljs).

I mostly stole the code from a [three.js example](https://threejs.org/examples/webgl_terrain_dynamic.html), since I thought it looked good.

### <a name="height-lookups">Fast heightfield lookup</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/ground_local.cljs).

I first thought I had to do bilinear interpolation, so I implemented that, but of course, it was just triangles in the mesh, no bilinear interpolation needed. So it's just finding the correct position in the heightfield and then interpolating the triangles, using math from [here](http://stackoverflow.com/questions/16077725/three-js-precision-terrain-collision).

### <a name="voxelization">Voxelization of 3D geometries</a>
First off, [here is a link to the source code](https://github.com/emnh/rts/blob/master/src.client/game/client/voxelize.cljs). Voxelization means to represent each part of a 3D geometry as a small box of the same size, rather than triangles of varying size. I used the approach described [here](http://drububu.com/miscellaneous/voxelizer/index.html). Well, it seems the author of that page has removed the description of the algorithm and just left the online voxelizer. Anyway, the algorithm is not too complicated: it runs entirely on the CPU (no GPU) and involves recursively subdividing each triangle in the 3D geometry until a constant threshold is hit, then check which box the triangle lies in and mark it as active / on. In addition, I did a flood fill to mark all interior boxes as on as well, since I wanted to explode the voxels and thus needed the inside filled. The algorithm is quite simple and slow, taking up to an hour to voxelize a simple 3D model, but I don't do it realtime, I just run it offline as a script with node.js over-night and save the voxels for the game to load, so it doesn't matter that much. If you want to do realtime voxelization you should look [here](https://developer.nvidia.com/content/basics-gpu-voxelization) instead.

Later I decided to add UV coordinates for texture mapping to each box relative to the whole 3D model so that I can texture each box as close to the original geometry as possible. I proceeded to use the same recursive subdivision of each triangle T, then for each corner V of the small triangle and for each corner C of the box which V resides in, I update the UV of the box corner C (really, the UV of the box which has this corner as top left corner) if the closest point P on the big triangle T containing V is the current closest to C, and also save T as the closest triangle to C. I update the UV to the interpolation of UVs from the corners of the subdivided triangle T from the original 3D model using barycentric coordinates and P. As an optimization, for each box corner I also maintain a set of seen triangles and skip any triangle that has been seen before, since we already know what the closest point of that triangle to the box corner is. It's a naive algorithm in that it's the first thing that popped into my head, even if I had a bit trouble understanding the code that I wrote before now, and I am sure there are better approaches, but at least it gave results resembling the texturing of the original triangle 3D model.

[Before voxelization screenshot:](https://emnh.github.io/rts-blog-screenshots/shots/before-voxelization.jpg)
![Before voxelization](https://emnh.github.io/rts-blog-screenshots/shots/before-voxelization.jpg)
[After voxelization screenshot:](https://emnh.github.io/rts-blog-screenshots/shots/after-voxelization.jpg)
![After voxelization](https://emnh.github.io/rts-blog-screenshots/shots/after-voxelization.jpg)

TODO: Add some illustrative schematics.
TODO: Describe how to edit and run the voxelization script using node.js from the commandline.

### <a name="explosions">Explosions</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/explosion.cljs).

TODO: Reread and explain how the GLSL code works.

Here is a [screenshot](https://emnh.github.io/rts-blog-screenshots/shots/explosions.jpg):
![screenshot](https://emnh.github.io/rts-blog-screenshots/shots/explosions.jpg)

Watch the [live demo here](https://emnh.github.io/voxel-explosions-demo/#game-test).

### <a name="magicstars">Magic stars</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/magic.cljs).

I thought it would look nice to have magic stars raining down on buildings to indicate they are under construction or some other property, perhaps being under a spell. The GLSL just renders quads facing the camera, billboards as they are called and moves them from a source according to a formula to the location of the voxelized boxes of the unit geometry.

[A screenshot follows:](https://emnh.github.io/rts-blog-screenshots/shots/magic-stars.jpg)
![Magic stars](https://emnh.github.io/rts-blog-screenshots/shots/magic-stars.jpg)

### <a name="healthbars">Health bars</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/overlay.cljs).

PS: There is a bug in this screenshot, where some health bars are hidden by unit geometry. Should be easy to fix by changing the z-order.

[A screenshot follows:](https://emnh.github.io/rts-blog-screenshots/shots/health-bars.jpg)
![Health bars](https://emnh.github.io/rts-blog-screenshots/shots/health-bars.jpg)

### <a name="water">Water</a>

I adapted the [WebGL water code from Evan Wallace](http://madebyevan.com/webgl-water/) to work with a terrain, although I was not successful in getting the caustics to render properly. Maybe I will try again later. The way to bounce off terrain was simply to return zero for the neighbouring cell which is terrain when averaging the 4 neighbours of a cell. Easy, right? :) I figured it out after studying the code of [this WebGL ripple tank](http://www.falstad.com/ripple/). I experimented with the shader first at [ShaderToy](https://www.shadertoy.com/view/XsByzK) but ended up using a different simpler version for the game.

[Here is a screenshot of the ShaderToy shader:](https://emnh.github.io/rts-blog-screenshots/shots/water-terrain-shader.jpg)
![ShaderToy terrain/water shader:](https://emnh.github.io/rts-blog-screenshots/shots/water-terrain-shader.jpg)

[Here is a screenshot of the water in my game](https://emnh.github.io/rts-blog-screenshots/shots/water.jpg)
![Game water](https://emnh.github.io/rts-blog-screenshots/shots/water.jpg)

### <a name="marquee">Marquee selection</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/selection.cljs).

I tried out 3 different approaches to get this working fast and accurate:
 - Subfrustum selection is fast, but inaccurate.
 - Projecting unit bounding boxes onto screen is slow.
 - Project unit bounding spheres onto screen is fast and accurate.
 
Check out these links and Stack Overflow questions for more details:
 - [Marquee Selection with Threejs](http://blog.tempt3d.com/2013/11/marquee-selection-with-threejs.html)
 - [StackOverflow: Three.js: How to detect the intersection of a 2D square and 3D object](http://stackoverflow.com/questions/27295415/three-js-select-tool-how-to-detect-the-intersection-of-a-2d-square-and-3d-obj)
 - [StackOverflow: Three.js: How to pick all objects in area](http://stackoverflow.com/questions/20169665/threejs-how-to-pick-all-objects-in-area)
 
[A screenshot follows:](https://emnh.github.io/rts-blog-screenshots/shots/marquee.jpg)
![Marquee selection](https://emnh.github.io/rts-blog-screenshots/shots/marquee.jpg)

### <a name="minimap">Minimap</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/minimap.cljs).

The minimap is just the same scene rendered from a fixed top down perspective, except units are toggled invisible and unit box markers are toggled visible and fixed at a certain height above max terrain elevation. The blue square with black border is created using [this technique](https://stemkoski.github.io/Three.js/Outline.html). In the following screenshot I zoomed the camera out so you can see that it's just mostly the same rendering for minimap. It seems fast enough currently, but it might be worth optimizing by caching the terrain and water rendering for the minimap later. You can't click the minimap yet, so I have a todo item there.

[A screenshot follows:](https://emnh.github.io/rts-blog-screenshots/shots/minimap.jpg)
![Minimap](https://emnh.github.io/rts-blog-screenshots/shots/minimap.jpg)

### <a name="mathbox">Attack vectors using MathBox</a>

[The source code is here](https://github.com/emnh/rts/blob/master/src.client/game/client/mathbox.cljs).

It might be overkill to include [MathBox](https://github.com/unconed/mathbox) in an RTS, but [line drawing in WebGL is hard](https://mattdesl.svbtle.com/drawing-lines-is-hard) and I wanted some vectors to show which unit is attacking which other unit. I used an [updated fork of MathBox for three.js r84](https://github.com/znah/mathbox/) that I found a link to on the [MathBox Google Group](https://groups.google.com/forum/#!forum/mathbox). It was a bit of a hurdle to get the MathBox context, coordinate and scaling system to work well with my existing three.js scene, but once it was working MathBox is really smooth and fun to work with. I used a GLSL shader to resample and animate the vector paths.

[A screenshot follows:](https://emnh.github.io/rts-blog-screenshots/shots/mathbox.jpg)
![Mathbox attack vectors](https://emnh.github.io/rts-blog-screenshots/shots/mathbox.jpg)
