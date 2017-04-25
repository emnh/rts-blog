# RTS Blog

I am creating a [real-time strategy (RTS) game in WebGL and ClojureScript](https://github.com/emnh/rts). This is my blog about the process. I made the mistake of not documenting my development initially, but I will start now (25.04.2017) and try to look back on the code I've already written. If you want to comment on this blog [open an issue](https://github.com/emnh/rts-blog/issues) per topic or find an existing one on the GitHub repository. I might attach issue comments to the blog later using [this process](http://donw.io/post/github-comments/).

## Index of blog posts / topics
 - [History](#history)
 - [Game plan](#gameplan)
 - [ClojureScript component architecture](#components)
 - [ClojureScript React library: Rum](#webui)
 - [Single-page application](#spa)
 - Resource downloading and progress manager
 - Bootstrapping ClojureScript in a web worker
 - Engine logic web worker separation and message passing
 - ClojureScript optimizations
 - Terrain
 - Fast heightfield lookup in ClojureScript
 - Voxelization of 3D geometries
 - Unit explosions using voxelized representation
 - Magic stars
 - Marquee selection
 - Water
 - Attack vectors using MathBox
 - Multiplayer?

## Blog posts / topics

### <a name="history">History</a>

Initially the game was written in JavaScript with Babel, the ES2015 and beyond to ES5 compiler. I switched (more or less complete rewrite) to ClojureScript because the mutable state everywhere was becoming a mess, for hot code reloading and because Babel compilation times were too high (I think on the order of 30 seconds, but I don't remember so well). I still haven't implemented feature parity with the original version, but I've been focusing on new features like fancy terrain and water instead and notably Starcraft 2 3D model support has been dropped. I suppose the Starcraft 2 on WebGL headline was sparking some of the initial interest, getting a [Hacker News submission](https://news.ycombinator.com/item?id=10205347) on the front page and around 13k views, but support for non-free models is by far not the primary goal of the project. Developing a strong and independent engine is.

Here is a screenshot of the version with free models:
![screenshot of rts-free](https://emnh.github.io/rts-blog-screenshots/shots/rts-free.jpg)

[Here is the live version](http://emh.lart.no/publish/rts-free.git/).

The version which downloaded SC2 models from another host is not working anymore since the host is not serving the files anymore, so all you can do is watch YouTube videos of it (or if you are brave check out the [legacy branch of rts](https://github.com/emnh/rts/tree/legacy) and try to get it to work):

 - [StarCraft 2 on WebGL](https://www.youtube.com/watch?v=PoPNrz2LUG0)
 - [Starcraft 2 on WebGL with colorful UV mapping](https://www.youtube.com/watch?v=EvhUteDp3o8)
 - [Banelings, banelings, banelings](https://www.youtube.com/watch?v=aqKsVelmeeI)

### <a name="gameplan">Game plan</a>

My RTS will at least contain the major engine components that an RTS needs. We'll see what it ends up as. It might become just an engine with a simple tower defense as a demo or something totally different, depending on technical hurdles and wild ideas coming up during development experiments. I am not sure whether I will support multiplayer yet, or whether I will make a series of single player tutorial steps where you write JavaScript or ClojureScript to control the AI of the game to overcome various scenarios (test cases), like CodingGame, but with fancier graphics and more complex rules similar to a commercial RTS. Currently I am leaning towards the latter, because it allows more freedom about simulation speed and no synchronization issues, but at the same time I want the game to be deterministic with a replay function, so I can't generate too much simulation data. It will be hard to tack on multiplayer support later, so I need to make the decision before I start implementing the engine logic beyond the graphics. Currently I've been focusing on the graphics. I defer naming my game until I know what it will be about. I detest having to repeat myself in strategy games building the base, so I want it to be able to be automated as well as avoid any stressful micro with AI scripts, but I still want the game to be able to have human interaction with scripts as executable in-game tasks. The game should be designed such that the best human strategy should win over any AI macro/micro helper scripts.

### <a name="components">ClojureScript component architecture</a>

I chose [Stuart Sierra's component framework](https://github.com/stuartsierra/component) for ClojureScript to implement the game components. This accomplishes basically the same as a dependency injection framework: all the dependencies, the wiring together of components, are specified at the top level and allows for more easy reconfiguration. A component is a record with a protocol (interface) for start and stop methods. Whenever you change a file, figwheel reloads the scripts for the page and using the component library I  restart by calling stop on each component in the system, in reverse dependency order, and then start on each one in forward order. In practice, for most heavy components, like the resource loader for textures and geometries, I do nothing in the stop handler and in the start handler I only load the resources if they haven't been loaded before. I only do a real stop and start for light components, and for the component which I am currently working on. I should have been more diligent about implementing proper stop handlers and a system for changing which components get reloaded and not, as I currently have to do a full page reload for most changes, so I'm losing hot reloading. I just got lazy about it. I need to do some refactoring to address the issue, but page reloads are not much slower currently than a figwheel reload. Refactoring for hot reloading will pay off if I am in the middle of a game and want to change something however, but I want to be able to serialize game state anyway, so perhaps saving to localStorage and page reload will do.

I made a helper macro for creating components called defcom that defines a component record along with its dependencies, since I will know them up front. It also adds :start-count, :stop-count and :started properties to each component. Most functions still take a component as input, which is basically a dictionary (in ClojureScript record form). So functions take dictionaries and return dictionaries, which makes it easy to add arguments.

All state is kept in one global dictionary, but again, I haven't been diligent about separating updatable atoms and JS objects from serializable state. I need to think more about this and refactor to address this as well, kind of like a virtual DOM that separates input state from UI objects for three.js, recreating and deleting UI objects as units are updated for example. I also look for inspiration from the [redux project](https://github.com/reactjs/redux), to be able to replace local component atoms with a grand central dispatch which updates the global nested state dictionary.

At one point I investigated a lot of alternatives to Stuart's component (list taken from [danielsz system](https://github.com/danielsz/system)):
- [[https://github.com/juxt/modular][modular]]
- [[https://github.com/palletops/leaven][leaven]] and [[https://github.com/palletops/bakery][bakery]]
- [[https://github.com/james-henderson/yoyo][yoyo]]
- [[http://docs.caudate.me/hara/#haracomponent][hara.component]]
- [[https://github.com/tolitius/mount][mount]]

They all have advantages and disadvantages, but in the end I decided not to switch (yet). I've been looking hard at [plumatic plumbing graph](https://github.com/plumatic/plumbing) to wire together dependencies lately, and I might switch to that, but I haven't felt like refactoring yet.

### <a name="webui">ClojureScript React library: Rum</a>
I chose to use a ClojureScript React library called [Rum](https://github.com/tonsky/rum). I don't remember why I chose this over [Om](https://github.com/omcljs/om) or [Reagent](https://github.com/reagent-project/reagent). I think it might have been this point from the Rum readme: "No enforced state model: Unlike Om, Reagent or Quiescent, Rum does not dictate where to keep your state. Instead, it works well with any storage: persistent data structures, atoms, DataScript, JavaScript objects, localStorage or any custom solution you can think of." Anyway the point of using any such React library over plain HTML for me is that hot reloading comes for free.
