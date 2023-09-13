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