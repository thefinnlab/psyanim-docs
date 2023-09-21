# Example Predator-Prey Experiment

Note: Full tutorial project + source code can be found [here](https://github.com/thefinnlab/psyanim-core-examples/tree/predator-prey-experiment).

## 1. Project Setup

In this tutorial, we'll create a full predator-prey experiment from start to finish, using a single scene definition as a template and creating several variations with different parameters.

To get started, let's create a directory called `my-predator-prey` and run the following commands from that directory to initialize our project:

```bash
npm init -y
npm install git+https://github.com/thefinnlab/psyanim-2.git git+https://github.com/thefinnlab/psyanim-cli.git
npx psyanim-cli --init
```

Next, let's create our scene definition by running the following command:

```bash
npx psyanim-cli -s PredatorPrey
```

## 2. Creating our Predator-Prey Scene

Open up `PredatorPrey.js` under your `/src` directory and update your `psyanim2` import statement so it has the following classes imported:

```js
import { 
    PsyanimConstants,

    PsyanimPredatorPrefab,
    PsyanimPredatorAgent,

    PsyanimPreyPrefab,
    PsyanimPreyAgent,

} from 'psyanim2';
```

Then update the exported scene definition so it contains the following scene definition:

```js
export default {
    key: 'PredatorPrey',
    wrapScreenBoundary: false,
    entities: [
        {
            name: 'predator',
            initialPosition: { x: 100, y: 100 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
                radius: 12, color: 0xff0000
            },
            prefab: { 
                type: PsyanimPredatorPrefab,
                params: {
                    maxChaseSpeed: 5,
                    maxWanderSpeed: 4.5,
                    boredomDistance: 250,
                    showDebugGraphics: true,
                    showDebugLogs: false,
                    subtlety: 30
                }
            },
            components: [
                {
                    type: PsyanimPredatorAgent,
                    params: {
                        target: {
                            entityName: 'prey'
                        }
                    }    
                }
            ]
        },
        {
            name: 'prey',
            initialPosition: { x: 700, y: 500 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
                radius: 12, color: 0x0000ff
            },
            prefab: { 
                type: PsyanimPreyPrefab,
                params: {
                    maxFleeSpeed: 10,
                    maxWanderSpeed: 5.0,
                    showDebugGraphics: true,
                    showDebugLogs: false,
                    safetyDistance: 250,
                }
            },
            components: [
                {
                    type: PsyanimPreyAgent,
                    params: {
                        target: {
                            entityName: 'predator'
                        }
                    }    
                }
            ]
        }
    ],
}
```

Notice we only have two agents here and we use prefabs to attach components to them.

The only `component parameters` we have to override are the `target` params of the `*Agent` components added by each `prefab`, because we can not add `entity` or `component` references directly to a `prefab parameters`.

Notice that we only override a subset of the `prefab` parameters for our `predator` and `prey` entities.  Whatever parameters we don't override will take on the parameter default values.

At this point, your `PredatorPrey.js` contents should look like [this](https://github.com/thefinnlab/psyanim-core-examples/blob/predator-prey-experiment/src/PredatorPrey.js).

## 3. Setting up multiple trials with parameter variations

Let's open up our `index.js` and start setting up our `jsPsych trials` with variations of certain parameters.

Since we'll be adding variations of the same scene definition to each of our trials, we need to clone our scene definition and modify it's parameters.  To do this, we'll need to update our imports to include `PsyanimUtils`:

```js
import { 
    PsyanimApp, 
    PsyanimJsPsychPlugin,
    PsyanimFirebaseClient,
    PsyanimUtils

} from 'psyanim2';
```

Next, we need to import the `PredatorPrey` scene definition:

```js
import PredatorPrey from './PredatorPrey';
```

Let's delete all of the code under the comment `Setup jsPsych Experiment` and replace it with the following:

```js
const jsPsych = initJsPsych();

let timeline = [];

// 'welcome' trial
timeline.push({
    type: htmlKeyboardResponse,
    stimulus: 'Welcome to the experiment.  Press any key to begin.'
});

// setup predator-prey parameter-sets
let nTrials = 6;
let subtletyParams = [0, 30, 60, 90, 120, 150];

// we're going to randomly assign spawn locations to predator and prey for each trial
let leftSpawnPoint = { x: 100, y: 100 };
let rightSpawnPoint = { x: 700, y: 500 };

let predatorSpawnPoints = [];
let preySpawnPoints = [];

let predatorColors = [];
let preyColors = [];

for (let i = 0; i < nTrials; ++i)
{
    let predatorSpawnsOnLeft = Math.random() > 0.5;

    if (predatorSpawnsOnLeft)
    {
        predatorSpawnPoints.push(leftSpawnPoint);
        predatorColors.push(0x000000);

        preySpawnPoints.push(rightSpawnPoint);
        preyColors.push(0xcccccc);
    }
    else
    {
        predatorSpawnPoints.push(rightSpawnPoint);
        predatorColors.push(0xcccccc);

        preySpawnPoints.push(leftSpawnPoint);
        preyColors.push(0x000000);
    }
}

// create the trials and add them to timeline
for (let i = 0; i < nTrials; ++i)
{
    // clone the original JsPsychIntegrationScene so we can vary its params w/o modifying the original
    let trialScene = PsyanimUtils.cloneSceneDefinition(PredatorPrey);

    // modify the scene key to be something unique and register the scene
    trialScene.key += '_subtlety_' + subtletyParams[i];

    PsyanimApp.Instance.config.registerScene(trialScene);

    // modify subtlety parameter for this scene
    let predatorDefinition = trialScene.entities
        .find(e => e.name === 'predator');
    
    let preyDefinition = trialScene.entities
        .find(e => e.name === 'prey');

    predatorDefinition.initialPosition = predatorSpawnPoints[i];
    predatorDefinition.shapeParams.color = predatorColors[i];

    preyDefinition.initialPosition = preySpawnPoints[i];
    preyDefinition.shapeParams.color = preyColors[i];

    predatorDefinition.prefab.params.subtlety = subtletyParams[i];

    timeline.push({
        type: PsyanimJsPsychPlugin,
        sceneKey: trialScene.key,
    });
}

// 'End' trial
timeline.push({
    type: htmlKeyboardResponse,
    stimulus: 'Congrats - you have completed your first experiment!  Press any key to end this trial.'
});

jsPsych.run(timeline);
```

At this point, your `index.js` file should look like [this](https://github.com/thefinnlab/psyanim-core-examples/blob/predator-prey-experiment/src/index.js).

Reload the experiment in your browser and you should have 6 `PredatorPrey` trial variations where the predator is sometimes on the left, sometimes on the right, and the `agent` on the left is always `black` while the `agent` on the right is always `gray`.

In this experiment, we're varying the predator's `subtlety` parameter in each trial, as defined in the `subtletyParams` array.

The basic workflow for setting up multiple trials as variations of the same scene definition is the following:

- Define all variables / parameters in arrays which will be used in each trial
- Loop over each trial parameter-set, cloning the original scene definition and modifying the parameters, including `scene key`.
- In the same loop, push the newly modified `scene definition` into our `jsPsych timeline`.

Note that the scene definitions are just Javascript objects, and thus, when we clone it, we receive a copy of the original as a Javascript object.

To modify simulation parameters for each trial, all we need to do is modify the javascript object, whose structure is that of a `Psyanim scene definition`.

**Congrats - you've built a predator-prey experiment where the predator's `subtlety` parameter is takes on different values in each trial, and the predator agent is sometimes on the left or right, but color is always fixed by starting location.**

## 4. Setting up `Mimic` trials with multiple parameter variations.

Let's setup some predator-prey `Mimic` trials, where the predator doesn't chase the prey directly, rather it chases an entity that is 'mimicking' the prey's behavior.

To do this, we will setup a predator-prey scene, same as before.  However, this time, the prey will be made 'invisible' by making it the same color as the background.

Moreover, we will add a third entity to the scene which is our `Mimic` entity.  This entity will 'Mimic' the movements of the prey, possibly offsetting it's position and orientation relative to the prey.

To start, run the following command in terminal to create our `PredatorPreyMimic.js` scene definition file:

```bash
npx psyanim-cli -s PredatorPreyMimic
```

Open `PredatorPreyMimic.js` and copy the contents of `PredatorPrey.js` into it.  Remember, this `Mimic` scene is just a variation of the standard `Predator-Prey` scene, so we can safely use this as a starting point.

In `PredatorPreyMimic.js`, rename the scene definition `key` field from `PredatorPrey` to `PredatorPreyMimic` and add `PsyanimMimic` to your `psyanim2` import statement at the top.

In the `prey` entity's `shapeParams` field, change the `color` field to `0xffff00` and add a `depth` field with a value of `0` as follows:

```js
...
        {
            name: 'prey',
            initialPosition: { x: 700, y: 500 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
                radius: 12, color: 0xffff00, depth: 0
            },
            ...
        }
...
```

We've made the `prey` agent `yellow` colored so it doesn't have as much contrast with the background, but we can still see it for debugging purposes.  Note that we should change this to white to make it invisible before we deploy our experiment to production.

The `depth` field changes the rendering order for the entity - if two entities overlap, the entity with the higher `depth` value in its `shapeParams` settings will be drawn on top.

We want the `Mimic` entity to be drawn on top of the `prey`, so we will set the `prey` to a depth value of 0 for now.

Now, let's add our third entity, the `Mimic` entity as follows to our `entities` array in our `scene definition`:

```js
...
        {
            name: 'preyMimic',
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
                radius: 12, color: 0x0000ff, depth: 1
            },
            matterOptions: {
                isSensor: true
            },
            components: [
                { 
                    type: PsyanimMimic,
                    params: {
                        target: {
                            entityName: 'prey',
                        },
                        xOffset: 0,
                        yOffset: 0,
                        angleOffset: 0
                    }
                }
            ]
        }
...
```

As you can see here, all we hage to do to make this `preyMimic` entity mimic the `prey` entity is to add a `PreyMimic` component to it and set it's `target` parameter to reference the `prey` entity.

Note that we made the `mimic` entity `blue` here, so it will stand out against the `white` canvas more than our `prey` and make it easier to observe.

Moreover, we set the `depth` field of the `mimic` to a value of `1`, so it will always be rendered on top of the `prey` in cases where they overlap in the world.

The `xOffset`, `yOffset` and `angleOffset` are optional, but essentially control the horizontal / vertical position offset and relative orientation of the `mimic` with respect to the `prey` as the prey moves forward/backward/left/right.

Your final `PredatorPreyMimic.js` file should look like [this](https://github.com/thefinnlab/psyanim-core-examples/blob/predator-prey-experiment/src/PredatorPreyMimic.js).

---

To get a better feel for these parameters, we'll setup multiple `Mimic` trials using this scene and different parameters for each trial.

Let's open up `index.js` and import the `PredatorPreyMimic` scene at the top of the file.

Then, right after we setup our previous `Predator-Prey` trials, but before we push the `End trail` into our timeline, let's add the following code for our 3 `Mimic` trials:

```js

/**
 *  Setup mimic scenes
 */
let nMimicTrials = 3;

let mimicTrialParams = [
    { xOffset: 0, yOffset: 0, angleOffset: 0 },
    { xOffset: 30, yOffset: 30, angleOffset: 0 },
    { xOffset: -300, yOffset: -200, angleOffset: 180 },
];

for (let i = 0; i < nMimicTrials; ++i)
{
    let trialScene = PsyanimUtils.cloneSceneDefinition(PredatorPreyMimic);

    // update scene key to be unique
    trialScene.key += '_' + i;

    // register this new scene with psyanim app
    PsyanimApp.Instance.config.registerScene(trialScene);

    // let's update the scene params
    let mimicEntity = trialScene.entities
        .find(e => e.name === 'preyMimic');

    let mimicComponent = mimicEntity.components[0];

    mimicComponent.params.xOffset = mimicTrialParams[i].xOffset;
    mimicComponent.params.yOffset = mimicTrialParams[i].yOffset;
    mimicComponent.params.angleOffset = mimicTrialParams[i].angleOffset;

    // add mimic scene to timeline
    timeline.push({ 
        type: PsyanimJsPsychPlugin,
        sceneKey: trialScene.key,
    });
}
```

Here, we setup 3 `Mimic` trials, modifying the `xOffset`, `yOffset` and `angleOffset` parameters in each one.

Recall we had to clone our `PredatorPreyMimic` scene, modifying it's `scene key` to make it unique and registering it with `PsyanimApp`.

Then, we can modify this cloned scene's parameters without affecting the original.

Once we've updated all the parameters for each trial, we push them into the timeline.

The 3 parameter-sets we've chosen demonstrate the following cases, in order:

- `Mimic` entity directly copying every single movement of the `prey` entity.
- `Mimic` entity directly copying every single movement of the `prey` entity, but with it's position offset by `30 pixels` in the horizontal and vertical directions.
- `Mimic` entity is copying the movements of the `prey` entity, but with both a `translational` and `rotational` offset together. (Note: a 180 degree `angleOffset` basically performs all the target entity's movements, but in the reverse direction)

**Great work!  If you reload your experiment in the browser, you should see your 3 new trials in action!**