# Entities and Components

**TODO: make sure you show in this section how to access custom phaser functionality in any component! want to show that we have the full power of phaser's APIs in our components!**

**TODO: you should have multiple scenes here where entities and components are reused.**

**TODO: you should have a section on prefabs here!**

## 1. What's a component?

To this point, we've learned that `scenes` are like a stage where characters and animations are presented to the user, and `entities` are *things* that live in `scenes`.

More specifically, `entities` are any objects in a scene that have a position, orientation and velocity which can be controlled either directly or via forces / accelerations.

They can also, optionally, have a visual representation in the scene.

In the previous tutorial, we were able to add logic to our scene which is executed every frame to move an entity (our 'player' entity) around in response to control input from the keyboard.

While adding logic directly to our scene is a powerful ability, it isn't easily portable to other scenes or entities.  

We certainly don't want scenes referencing other scenes, b.c. then all of our scenes would be tightly coupled to each other... and since experiments are made up of scenes, our experiments could become tightly coupled to each other.

The solution to this problem is what we call a `Psyanim Component`.

A `PsyanimComponent` is a modular object that can be attached to an entity, and which has it's own user-defined `update(t, dt)` method which runs every simulation frame.

`PsyanimComponents` can be attached to any entity and any scene, thus making it easy to reuse and a better place for most of our AI & animation logic than directly in the scene itself.

As a general rule of thumb, if you have logic that is specific to one scene only and doesn't need to be reusable across scenes or different entities, it's probably OK to put that logic directly into the `PsyanimScene` definition itself.

If you do need to reuse a bit of behavior or game logic across multiple entities or scenes, it's best to abstract that out into a `PsyanimComponent`.

In the following tutorial, we'll explore `PsyanimComponents` in depth.

## 2. Components in action: creating a simple game in `Psyanim 2.0` with multiple levels

To build a more intuitive feel for the power of `PsyanimComponents` and how best to utilize them, we can create a simple game consisting of 3 levels.

The 'levels' will be implemented as `PsyanimScenes`.

In this game, we'll build a `Player Controller` component that can be reused across scenes on different entities.

### a. Setup your npm project and install psyanim-2

Let's start by opening a terminal (or powershell if that's your thing :) and creating a directory called `hello-psyanim-components`.

Navigate to the directory we just created and initialize an npm project with:

```bash
    npm init -y
```

Install psyanim-2 and psyanim-cli with:

```bash
    npm install git+https://github.com/thefinnlab/psyanim-2.git git+https://github.com/thefinnlab/psyanim-cli.git
```

Now, we can initialize our psyanim experiment with:

```bash
    npx psyanim-cli --init
```

### b. Setup three 'levels' as a scenes

Let's go ahead and delete the `EmptyScene.js` under `./src/`.

We can create the our scenes for our 'levels' with the following command:

```bash
    npx psyanim-cli -s MyLevel1,MyLevel2,MyLevel3
```

You should see the scene files for those levels show up under `./src/`.

Then, open up `index.js` in the `./src/` directory.

We won't need `jsPsych` here, so go ahead and delete everything in your index.js and we'll build it from scratch as an exercise.

Copy the following code into your `index.js` file:

```js
    import { PsyanimApp } from 'psyanim2';

    import MyLevel1 from './MyLevel1';
    import MyLevel2 from './MyLevel2';
    import MyLevel3 from './MyLevel3';

    /**
     *  Setup Psyanim and PsyanimJsPsychPlugin
     */
    PsyanimApp.Instance.config.registerScene(MyLevel1);
    PsyanimApp.Instance.config.registerScene(MyLevel2);
    PsyanimApp.Instance.config.registerScene(MyLevel3);

    PsyanimApp.Instance.run();
```

Open up each level's scene file under `./src/` and delete the `init()`, `preload()`, and `update(t, dt)` methods in each one.

We will be putting all of our game logic & behaviors into `PsyanimComponents` we'll be creating, so the only method we'll need in our scene is `create()`, which is where we'll declare all of our entities and attach components to them.

### c. Our first component: A level loader

Right now, our game has 3 levels, but the first level to load is the first scene we registered with `PsyanimApp`, which is `MyLevel1`.

Once our `MyLevel1` scene loads, it will remain the active scene in our game until we load a different level from our code.

Since we want to be able to load a level from any scene, it's not ideal to copy / paste our level loading code into every single scene we create.

Instead, let's create a `PsyanimComponent`, called `MyLevelLoader`, which can be added to any scene which allows us to load a level via a keypress.

To create the component in our project, simply navigate to our project directory in your terminal and run:

```bash
    npx psyanim-cli --component MyLevelLoader
```

Open up `MyLevelLoader.js` under `./src/` and you'll see the core of a `PsyanimComponent` class is very minimal, fundamentally only requiring a `constructor()` and an `update(t, dt)` method in its interface.

### TODO: walk through the basic setup of component that can load level by pressing '1', '2', or '3' alpha keys, and when each level is loaded, the scene loader prints out the scene.key to the console so we know what scene we're in.

### TODO: you may want to split up the sections in a way that more of them are exposed via the sidebar navigation