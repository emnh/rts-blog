# RTS Blog

I am creating a real-time strategy (RTS) game in WebGL and ClojureScript. This is my blog about the process. I made the mistake of not documenting my development initially, but I will start now (25.04.2017) and try to look back on the code I've already written.

## Index of blog posts / topics
 - [History](#history)
 - [Game plan](#gameplan)
 - ClojureScript component architecture
 - Resource downloading and progress manager
 - Bootstrapping ClojureScript in a web worker
 - Engine web worker separation and message passing
 - ClojureScript optimizations
 - Terrain
 - Fast heightfield lookup in ClojureScript
 - Marquee selection
 - Water
 - Attack vectors using MathBox
 - Initial thoughts about multiplayer

## Blog posts / topics

### <a name="history">History</a>

Initially the game was written in JavaScript. I switched (more or less complete rewrite) to ClojureScript because the mutable state everywhere was becoming a mess, for hot code reloading and because Babel compilation times were too high (I think on the order of 30 seconds, but I don't remember so well). I still haven't implemented feature parity with the original version, but I've been focusing on new features instead and notably Starcraft 2 3D model support has been dropped. I suppose the Starcraft 2 on WebGL headline was sparking some of the initial interest, getting a [Hacker News submission](https://news.ycombinator.com/item?id=10205347) on the front page and around 13k views.

Here is a screenshot of the version with free models:


### <a name="intro">Game plan</a>

My RTS will at least contain the major engine components that an RTS needs. We'll see what it ends up as. It might become an engine with a simple tower defense or something totally different, depending on technical hurdles and wild ideas coming up during development experiments. I am not sure whether I will support multiplayer yet, or whether I will make a series of single player tutorial steps where you write JavaScript or ClojureScript to control the AI of the game to overcome various scenarios (test cases), like CodingGame, but with fancier graphics and more complex rules similar to a commercial RTS. Currently I am leaning towards the latter, because it allows more freedom about simulation speed and no synchronization issues, but at the same time I want the game to be deterministic with a replay function. I defer naming my game until I know what it will be about. I detest having to repeat myself in strategy games building the base, so I want it to be able to be automated.
