# Psyanim 2.0: jsPsych Integration

Note: Full tutorial project + source code can be found [here](https://github.com/thefinnlab/psyanim-overview-tutorials/tree/jspsych-integration).

## 1. Psyanim-jsPsych Integration

`Psyanim 2.0` is designed as a standalone [Phaser](https://phaser.io/) app which renders to an `HTML Canvas`, as any other `Phaser` app does.

In order to use `Psyanim 2.0` in [jsPsych](https://www.jspsych.org/), a `jsPsych plugin` was developed: `PsyanimJsPsychPlugin`.

In this tutorial, we'll walk through how to use a `Psyanim scene` as a `jsPsych trial`, as well as how to create variations of the same scene in different trials, each with their own independent sets of parameters for the same scene.

## 2. Setting up our test scene

This tutorial picks up where the previous tutorial, `Entity Prefabs`, leaves off.

Let's create our `JsPsychIntegrationScene` by running the following command in the terminal:

```bash
npx psyanim-cli -s JsPsychIntegrationScene.js
```

Go ahead and open up `JsPsychIntegrationScene.js` in you `/src` directory and replace it's contents with the following:

```js
import 
{
    PsyanimConstants,
    PsyanimWanderAgentPrefab
} from 'psyanim2';

export default {

    key: 'Wander Test',
    entities: [
        {
            name: 'agent',
            instances: 20,
            initialPosition: 'random',
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
                base: 16, altitude: 32, 
                color: 0xffc0cb            
            },
            prefab: {
                type: PsyanimWanderAgentPrefab,
                params: {
                    debug: false,
                    radius: 150,
                    maxSpeed: 3,
                    maxAcceleration: 0.03,
                    maxAngleChangePerFrame: 10
                }
            }
        }
    ]
};
```

This is just a scene where we create 20 `Wander Agent` entities using a `PsyanimWanderAgentPrefab`.

We are able to control how many entities of this type we want in the scene by using the `instances` field of the `entity definition`.

We will create multiple trials, each with variations of this same scene definition in the next step.

## 3. Setting up jsPsych trials in index.js

There are many changes made to `index.js` for this tutorial, since it's a more advanced usage of Psyanim 2 than previously, so instead of walking through all the changes snippet by snippet, we will first just copy the code from this final `index.js` into our own `index.js`.

**So, go ahead and copy the code from [here](https://github.com/thefinnlab/psyanim-overview-tutorials/blob/jspsych-integration/src/index.js) into your index.js.**

- Pro-tip: Github's web client has a little 'Copy raw file' button on the top-right of any file to copy it's contents.

Reload your app in the web browser and you should see 20 triangle-shaped agents wandering about randomly in three different trials!

- In the first trial, the agents are green and wandering about rather slowly.

- In the second trial, the agents are purple and wandering about at the same pace as the first trial.

- In the third trial, the agents are a 'pinkish' color and wandering about at a significantly faster speed than the first two trials.

The important thing to note about these trials is that they were all created from the *same scene definition*!

## 4. The Anatomy of the `PsyanimJsPsychPlugin`

The main learning curve in `Psyanim 2.0` is understanding how to build interactive content in the `Psyanim scene definitions` or writing custom `Psyanim components`.

Once you've become comfortable with those two things, using `Psyanim 2.0` with jsPsych should be mostly a breeze.

---

Let's take a moment to walk through some of the fundamentals of using the `PsyanimJsPsychPlugin` in a `jsPsych experiment`.

For starters, note that any `Psyanim scene definition` must be registered with `PsyanimApp.Instance.config.registerScene()` before it can be referenced by `scene key` from a `jsPsych trial`.

Also note how we set the `PsyanimApp canvas` to be invisible at the start of the experiment.  This is desirable in most cases, as we may not have a `Psyanim scene` as the first jsPsych trial.

All of the `jsPsych` usage is the same as any other experiment, as described in the [docs](https://www.jspsych.org/).

The only main structural change we've made from our previous `tutorial` is that our jsPsych `timeline` is now declared as a named variable and we are pushing trials into it before passing it to `jsPsych.run()` at the bottom of the file.

Adding a `jsPsych trial` using the `PsyanimJsPsychPlugin` is very simple, with only two fields in the trial definition object: 

- `type`: the standard `jsPsych` trial definition `type` field, which specifies the type of `jsPsych plugin` this trial will use
- `sceneKey`: this is the `scene key` for any `Psyanim scene definition` that's been registered with `PsyanimApp`

And that's all there really is to the `PsyanimJsPsychPlugin` interface!

It's not much more complex than using any other `jsPsych plugin`. 

## 5. Creating trials with parameters variations

Notice we added the `PsyanimUtils` class to our imports.

This class has a helpful utility function called `cloneSceneDefinition` that can create a proper deep copy of any `Psyanim scene definition` for us.

We will use this deep-copy ability of the `cloneSceneDefinition` function to make independent variations of the original `scene definition` for different `trials`, so we can vary `trial parameters` in each one without affecting the others.

Let's now turn our attention to where we setup the trials using our `JsPsychIntegrationScene`:

```js
// JsPsychIntegrationScene trials - here we add multiple trials as variations of the same scene
let nTrials = 3;

let trialInstances = [3, 6, 20];
let trialAgentColors = [ 0x32CD32, 0xBC13FE, 0xFFC0CB ];

let trialParams = [
    {
        radius: 150,
        maxSpeed: 3,
        maxAcceleration: 0.03,
        maxAngleChangePerFrame: 10
    },
    {
        radius: 150,
        maxSpeed: 3,
        maxAcceleration: 0.03,
        maxAngleChangePerFrame: 10
    },
    {
        radius: 50,
        maxSpeed: 3,
        maxAcceleration: 0.2,
        maxAngleChangePerFrame: 20
    }
];

for (let i = 0; i < nTrials; ++i)
{
    // clone the original JsPsychIntegrationScene so we can vary its params w/o modifying the original
    let trialScene = PsyanimUtils.cloneSceneDefinition(JsPsychIntegrationScene);

    // modify the scene key to be something unique and register the scene
    trialScene.key += '_' + i;

    PsyanimApp.Instance.config.registerScene(trialScene);

    // modify any trial parameters we want to change
    trialScene.entities[0].instances = trialInstances[i];
    trialScene.entities[0].shapeParams.color = trialAgentColors[i];
    trialScene.entities[0].prefab.params = trialParams[i];

    // add it to the jsPsych timeline!
    timeline.push({
        type: PsyanimJsPsychPlugin,
        sceneKey: trialScene.key
    });
}
```

Recall that, in a previous step, we created our `JsPsychIntegrationScene` as just a bunch of wandering agents.

Here, we need to create 3 `trials` as variations of this same `scene definition`, but with different `parameters` for each `trial`.

The basic procedure to accomplish this is:

- Define arrays corresponding to the `parameter-sets` for each `trial`.
- Loop over the number of `trial variations` you want to create from the `scene definition`, creating a copy of the `scene definition` to add to a `trial` in our `timeline`
- Before adding the new copy of the `scene definition` to a `trial` in our `timeline`, update the relevant parameters with those from our pre-defined `parameter-sets`

Note that we use `PsyanimUtils.cloneSceneDefinition()` to create a copy of our `JsPsychIntegrationScene`.

We must use this utility function to create copies of `scene definitions` because we want a *deep-copy* that we can modify without affecting the others.

And that's all there is to it!  Great work - you've just created multiple `jsPsych trials` as variations of the same `Psyanim scene definition`!