# Entity Prefabs

## 1. What's an `Entity Prefab`?

**An `Entity Prefab` is like a blueprint that captures the necessary components for creating an entity with specific state and behaviors.**

In addition to capturing the necessary components, an `Entity Prefab` should handle the component configuration and can expose custom configuration parameters, providing a simpler interface for entity creation than the dealing with the individual components themselves.

We've seen, in the previous tutorials, that `entities` are objects that live in a `scene`.

We've also learned that we can add state and behavior to an `entity` by attaching `components` to it.

The simplest `entities` may have no `components` or maybe just 1 `component` attached to them.  For these `entities`, it's easy enough to create more of them simply by attaching the same type of `component` to another `entity` in the same `scene` or in other `scenes`.

However, as soon as we start creating & using agents with more complex behaviors requiring many components, such as the `Playfight`, `Predator`, or `Prey`, attaching all the necessary `components` and configuring them all properly can become unecessarily repetitive and cumbersome.

`Entity Prefabs` provide a means to hide this complexity and minimize the amount of repetitive setup code when adding agents composed of many components to your scene. 

## 2. Complex entities: Building a playfight behavior from components

**To understand the problem better, let's create an experiment with a simple `playfight scene` with two agents executing our `Playfight` algorithm.**

At it's core, the `Playfight` algorithm involves two agents wandering about the world randomly, and at specified intervals, they attack by charging at each other until they make contact, at which point they return to wandering until the next time to attack.

**a. Let's create a directory called `hello-entity-prefabs` and setup our experiment:**

Navigate to the `hello-entity-prefabs` directory and run the following commands to initialize our experiment:

```bash
npm init -y
npm install git+https://github.com/thefinnlab/psyanim-2.git git+https://github.com/thefinnlab/psyanim-cli.git
npx psyanim-cli --init
```

We can go ahead and delete the `EmptyScene.js` and then create our `PlayfightScene.js` with the following command:

```bash
npx psyanim-cli -s PlayfightScene
```

Open up index.js and replace its contents with the following:

```js
import { PsyanimApp } from 'psyanim2';

import PlayfightScene from './PlayfightScene';

/**
 *  Setup Psyanim and PsyanimJsPsychPlugin
 */
PsyanimApp.Instance.config.registerScene(PlayfightScene);

PsyanimApp.Instance.run();
```

**b. Let's implement our `PlayfightScene` by adding all the necessary entities and components:

Let's open up `PlayfightScene.js` and ...