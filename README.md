# RTS Blog

I am creating a [real-time strategy (RTS) game in WebGL and ClojureScript](https://github.com/emnh/rts). This is my blog about the process. I made the mistake of not documenting my development initially, but I will start now (25.04.2017) and try to look back on the code I've already written. If you want to comment on this blog [open an issue](https://github.com/emnh/rts-blog/issues) on the GitHub repository.

## Index of blog posts / topics
 - [History](#history)
 - [Game plan](#gameplan)
 - ClojureScript component architecture
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

My RTS will at least contain the major engine components that an RTS needs. We'll see what it ends up as. It might become just an engine with a simple tower defense as a demo or something totally different, depending on technical hurdles and wild ideas coming up during development experiments. I am not sure whether I will support multiplayer yet, or whether I will make a series of single player tutorial steps where you write JavaScript or ClojureScript to control the AI of the game to overcome various scenarios (test cases), like CodingGame, but with fancier graphics and more complex rules similar to a commercial RTS. Currently I am leaning towards the latter, because it allows more freedom about simulation speed and no synchronization issues, but at the same time I want the game to be deterministic with a replay function, so I can't generate too much simulation data. It will be hard to tack on multiplayer support later, so I need to make the decision before I start implementing the engine. Currently I've been focusing on the graphics. I defer naming my game until I know what it will be about. I detest having to repeat myself in strategy games building the base, so I want it to be able to be automated as well as avoid any stressful micro with AI scripts, but I still want the game to be able to have human interaction with scripts as executable in-game tasks.
