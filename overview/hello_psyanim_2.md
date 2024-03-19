# <ins>Getting Started with Psyanim 2.0</ins>

## 1. Introduction and Installation

***Pre-requisites: Requires [NodeJS](https://nodejs.org/en) v18.16.0+, an up-to-date installation of [Git](https://git-scm.com/), and [Psyanim-CLI](https://github.com/thefinnlab/psyanim-cli.git) installed globally.***

***You'll need to make sure you have `read` access to our `psyanim-2` and `psyanim-cli` private git repos***

`Psyanim 2.0` is a framework for creating browser-based psychology / cognitive science research experiments involving 2D procedural animation.

`Psyanim-CLI` is a tool for creating & managing experiment projects and assets in `Psyanim 2.0`.

If not installed already, you can install psyanim-cli package globally from git repo via npm with the following command:

```bash
npm install -g git+https://github.com/thefinnlab/psyanim-cli.git
```

## 2. Creating A New Project & First Scene

In a terminal, create a directory called `hello-psyanim2` and run the following command from that directory:

```bash
psyanim init
```

This command will create a blank npm project, install psyanim-2 & it's dependencies, and then setup a local git repo to keep things under version control.

Let's create a new scene by running the following command:

```bash
psyanim template:predpreyv2 -o ./src/scenes
```

This command creates a new type of experiment `asset` called a `scene definition` under `./src/scenes/HelloPredatorPreyV2.js`.

This particular scene definition is a `template` which already has two `agents` setup, where one behaves as a `predator` and the other behaves as a `prey`.

Let's add this scene definition to our `jsPsych experiment` now.

Open up `index.js` under the `./src` directory and add the following import after the rest of the imports:

```js
import PredatorPreyV2 from './scenes/PredatorPreyV2.js';
```

You can delete the `EmptyScene` import statement before that, as well as the `EmptyScene.js` file it imported.

Scroll down to the line where we create a variable called `emptySceneTrial` and delete that whole trial, including the line where it's pushed into the `jsPsych timeline`.

The lines to remove are:

```js
// psyanim empty scene trial
let emptySceneTrial = new PsyanimJsPsychTrial(EmptyScene, EmptyScene.key);
// emptySceneTrial.saveTrialMetadata = true;
// emptySceneTrial.addExtension(PsyanimJsPsychDataWriterExtension);

timeline.push(emptySceneTrial.jsPsychTrialDefinition);
```

Next, in the location where the `emptySceneTrial` previously was, we'll add our `PredatorPreyV2` scene definition to a trial in our jsPsych timeline with the following code:

```js
let predatorPreyV2Trial = new PsyanimJsPsychTrial(PredatorPreyV2, PredatorPreyV2.key);

timeline.push(predatorPreyV2Trial.jsPsychTrialDefinition);
```

Now, in a separate terminal, run the following command from the project root directory to start a `watch` service that rebuilds your code any time you change a file:

```bash
npm run watch
```

In another terminal, run the following command to start a local development server for testing:

```bash
npm run serve
```

In a Chrome Web Browser, nagivate to `localhost:3000` and you should see the `jsPsych` experiment's first trial.

Using the `enter` key, progress to the next trial, which is our `predator-prey v2` trial.

You should see two circular shaped agents in this trial, where the predator occasionallys turns red and starts pursuing the other agent.

The agent being pursued is the prey, and it will turn blue/yellow when it attempts to evade the predator.

When you are done watching the predator-prey interactions, you can progress through the rest of the trials using the `enter` key.

Great work - you've successfully setup a `Psyanim-2` experiment with a `predator-prey v2` trial!

## 3. Core Concepts: Project Structure

An `Psyanim-2` experiment project is composed of of many files.

However, except in advanced usage, there are only a few key files a researcher will be concerned with:

- <ins>*index.js*</ins>: The main entry point for the `psyanim-2` experiment project. Most configuration happens here.
- <ins>*scene definitions*</ins>: These scene definitions define the `psyanim-2 scenes` that are used in trials.
- <ins>*'/dist' directory*</ins>: This directory contains the fully bundled `psyanim-2` experiment, ready for deployment.

For `Google Firebase Firestore` database integration, the following file must be supplied by the user:

- <ins>*firebase.config.json*</ins>: This file contains authentication information necessary for a deployed `psyanim-2` experiment to access a particular firestore instance

Aside from these files, there are some files that researchers may be interested in, but mostly in advanced use cases:

- <ins>*index.html*</ins>: This is the actual HTML file for the page the experiment runs on.  This is mostly controlled by `psyanim-2` and `jsPsych`, so it shouldn't need to be edited directly during basic usage.
- <ins>*package.json & package-lock.json*</ins>: These are for package management, and should mostly be updated automatically.
- <ins>*other firebase\* files*</ins>: These are related to `psyanim-2`'s `firebase` integration, and should be left alone.
- <ins>*webpack.config.js*</ins>: This file controls the behavior of `webpack` and should only be modified by advanced users.

## 4. Core Concepts: Psyanim Scenes

`Psyanim 2.0` is built on top of [Phaser 3](https://phaser.io/) and has a flexible [component architecture](https://gameprogrammingpatterns.com/component.html).

In `Psyanim 2.0`, everything that is seen in the world lives in a `PsyanimScene`, which is an abstraction for the 2D world we are simulating.

The `PsyanimScene` acts as a container for `PsyanimEntity` objects.

We create `scene definitions` as Javscript objects, each in their own file.

Let's open up the `PredatorPreyV2.js` file we created in the first section and inspect it's contents more closely to see what fundamental elements a `PsyanimScene` is composed.

The `scene definition` object has the following fields:

- `key`: a key that uniquely identifies this particular scene in `Psyanim-2`
- `wrapScreenBoundary`: a flag that specifies whether or not the entity can cross the world (canvas) boundaries
- `entities`: an array defining what `entities` exist in this scene to start out

---

Any object that exists in a scene, regardless of its visual representation, is called a `PsyanimEntity`.

A `PsyanimEntity` acts a container for `PsyanimComponents`.  

More technically, it's an abstraction for anything that exists in a Psyanim Scene with a particular location, rotation, and (optionally) a visual representation which can have physics applied to it.

Entities may or may not have a visual representation in the scene.

Moreover, entities alone do not have any logic or behaviors.  While entities have no user-defined state, they do have a position, orientation and velocity in the world, and can have forces / accelerations applied to them.

Any object in your simulation that needs a position or orientation, a visual representation in the scene, or needs to have physics applied to it, should be added to the scene as an entity.

In `PredatorPreyV2.js`, there are only two entities configured in the scene definition: the 'predator' entity and 'prey' entity.

Each `entity` is added to the scene as a Javascript Object in the `entities` array of the scene definition object.

Each `entity definition` object in `PredatorPreyV2.js` has 4 fields: 

- `name`: a name that uniquely identifies it amongst the other entities in the scene.
- `initialPosition`: vector specifying where entity is initially located in 2D space
- `shapeParams`: object specifies the shape & color of the entity, or whether it's visible
- `components`: an array of `component definition` objects, each of which can add custom logic to be executed

In general, an `entity definition` only requires a `name` field.  All the other ones are optional.

---

All user-defined state and behaviors are encapsulated in `PsyanimComponents`, which are reusable scripts that can be attached to a `PsyanimEntity` and offer hooks into the real-time update loop of each scene.

In `PredatorPreyV2.js`, we see that each entity only has 1 `component` attached to each of them, as specified by the `components` array for each entity.

For example, the `component definition` for the finite-state machine containing the AI decision-making logic for our `predator` agent entity looks like the following:

```js
{
    type: PsyanimPredatorFSM,
    params: {
        target: {
            entityName: 'prey'
        },
        debugLogging: false,
        debugGraphics: true
    }
}
```

Note that each `component definition` object can accept parameters via the `params` object.

---

When building experiments using [jsPsych](https://www.jspsych.org/), we can run `trials` using the `PsyanimJsPsychPlugin` where each trial runs an instance of a `PsyanimScene`.

In `index.js` of the experiment we created in the previous section, you will see references to the `PsyanimJsPsychPlugin` and `PsyanimJsPsychTrials`.

We will discuss these in greater detail in later sections, but for now, it's enough to know that they are essentially the `glue` between `Psyanim-2` and a `jsPsych experiment`.

## 5. Adding Template Scenes

TODO: in this section, let's add some extra template scenes to our project and test with them