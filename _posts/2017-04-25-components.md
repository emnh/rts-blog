---
layout: post
title: ClojureScript component architecture
---

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


