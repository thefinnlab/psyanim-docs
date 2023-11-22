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
npx psyanim-cli -s JsPsychIntegrationScene
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

## 6. Trial end conditions

`Psyanim 2.0` provides 3 types of conditions upon which a trial will end:

- `Duration-based`
- `Key Press`
- `Player Contact`

The `duration-based` `trial end condition` uses a configurable timeout.  Once the configured time expires from trial start, the jsPsych trial ends.

We can also configure the trial to end when any keyboard key is pressed.

We can also setup the trial to end when the `player's entity` contacts another entity in the scene.

Finally, we can employ any or all of these trial end conditions together.

When using multiple trial-end conditions together, whichever condition is met first will trigger the end of the trial.

The following sections will explore how to use these `trial end conditions` in practice.

## 7. Ending trials by key press and duration

To see how to use the `duration-based` and `key press` trial end conditions, let's open up our index.js and peek at how our `Wander Agent` scenes use them.

The first thing we'll see is an array of `trialDurations` containing the `duration parameter`, in milliseconds, for each of our 3 `Wander Agent` trials.

We see the following line right before the line where we declare the `trialInstances` array:

```js
let trialDurations = [ 5000, 10000, 15000 ];
```

Next, in the `for-loop` where we push each `Wander Agent` trial definition into our `timeline` array, we see the `trial definition` includes the `duration` and `endTrialKeys` parameters:

```js
    // add it to the jsPsych timeline!
    timeline.push({
        type: PsyanimJsPsychPlugin,
        sceneKey: trialScene.key,
        duration: trialDurations[i],
        endTrialKeys: [' ', 'enter', 'h']
    });
```

Recall that there are 3 `Wander Agent` trials.  Here we are setting the `duration` parameter for each one to 5000, 10000, and 15000 milleconds, respectively.

We also add an `endTrialKeys` parameter which ends the trial if any of the following keys are pressed:

- Space-Bar
- Enter Key
- Lowercase 'h' key

These 3 `Wander Agents` trials will end when either of these conditions are met (duration time elapses or any of the configured keys are pressed).

By this point, the code in your `index.js` should look like [this](https://github.com/thefinnlab/psyanim-overview-tutorials/blob/jspsych-integration/src/index.js).

Reload your experiment in your browser to see this in action!

## 8. Ending trials based on player contact

To see how to use the `Player contact` trial end condition, let's modify our `InteractiveEvadeAgent.js` scene definition, adding some walls and making contact with certain entities end the trial.

Before we do that, if you look in your `index.js`, you should see that `endTrialOnContact` is set to `true` for the `InteractiveEvadeAgent` trial definition.

This flag in the `trial definition` is used to control whether or not the trial will end on player contact with any configured entities in the scene.

It can, optionally, be used in conjunction with the `duration` and `key press` trial end conditions.

In order for this to work, we'll need to make sure there's a `PsyanimJsPsychPlayerContactListener` in the scene and that the `player entity` has a `PsyanimSensor` attached to it.

The `PsyanimJsPsychPlayerContactListener` component notifies the `PsyanimJsPsychPlugin` when player contact has occured in the scene with an entity of interest.

The `PsyanimSensor` we attach to the player encapsulates the actual `matter-js` physics body that is used for `collision detection` in our `scene`, and thus must remain attached to the player entity, so it will share the same position and rotation in the world.

Open the scene definition file and add the `PsyanimSensor` and `PsyanimJsPsychPlayerContactListener` components to your `psyanim2` imports as follows:

```js
import 
{ 
    PsyanimConstants,
    PsyanimPlayerController,
    PsyanimEvadeAgentPrefab,
    PsyanimEvadeAgent,
    PsyanimSensor,
    PsyanimJsPsychPlayerContactListener
    
} from 'psyanim2';
```

Next, let's add 4 wall boundaries at the `north`, `south`, `east` and `west` sides of our world by adding the following entity definitions to our `entities` array:

```js
...
        {
            name: 'northWall',
            initialPosition: { x: 400, y: 15 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.RECTANGLE,
                width: 720, height: 30, color: 0xff0000
            },
            matterOptions: {
                isStatic: true,
            }
        },
        {
            name: 'southWall',
            initialPosition: { x: 400, y: 585 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.RECTANGLE,
                width: 720, height: 30, color: 0xff0000
            },
            matterOptions: {
                isStatic: true,
            }
        },
        {
            name: 'westWall',
            initialPosition: { x: 20, y: 300 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.RECTANGLE,
                width: 40, height: 600, color: 0x00ff00
            },
            matterOptions: {
                isStatic: true,
            }
        },
        {
            name: 'eastWall',
            initialPosition: { x: 780, y: 300 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.RECTANGLE,
                width: 40, height: 600, color: 0x00ff00
            },
            matterOptions: {
                isStatic: true,
            }
        },
...
```

These wall boundaries are `red` and `green` colored.  We want to setup the scene so the trial ends if the player touches the `red` walls or the `evade agent`.

To accomplish this, we simply need to add a `PsyanimSensor` and a `PsyanimJsPsychPlayerContactListener` to our `player` agent as follows:

```js
        {
            name: 'player',
            initialPosition: { x: 400, y: 300 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
                radius: 12,
                color: 0x0000ff
            },
            components: [
                { 
                    type: PsyanimPlayerController,
                    params: {
                        speed: 12
                    }
                },
                {
                    type: PsyanimSensor,
                    params: {
                        bodyShapeParams: {
                            radius: 18,
                            shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE
                        }
                    }
                },
                {
                    type: PsyanimJsPsychPlayerContactListener,
                    params: {
                        sensor: {
                            entityName: 'player',
                            componentType: PsyanimSensor
                        },
                        targetEntityNames: [ 
                            "agent1",
                            "northWall",
                            "southWall"
                        ]
                    }
                }
            ]
        },
```

The `PsyanimSensor` component's params just specify the size and shape of the sensor.  Here, we make it a `circle` shape of `radius` '18'.

In order for the `PsyanimJsPsychPlugin` to be notified when a player agent in a `scene` makes contact with something that should end the `jsPsych trial`, we need to add a `PsyanimJsPsychPlayerContactListener` to any entity in the scene.

For convenience, we can just add the `PsyanimJsPsychPlayerContactListener` component to our `player` entity as shown in the previous code snippet.

The only two parameters we need to configure for this component are the `sensor` reference and the `targetEntityNames`.

The `sensor` reference is simply a reference to the sensor that we want to listen for collision events from, which is the `PsyanimSensor` component of our `player` agent.

The `targetEntityNames` parameter expects a list of `entity` names for which colliding them would cause the `jsPsych trial` to end.

Notice that we've increased the `PsyanimPlayerController` component's `speed` parameter to `12` also.  This will give us the opportunity to 'catch' the `evade agent` as it tries to escape from the `player` entity.

To make the 'chase' a little easier, let's also slow down the `evade agent` by reducing the `maxSpeed` parameter of `agent1`'s `PsyanimEvadeAgentPrefab` as follows:

```js
        {
            name: 'agent1',
            initialPosition: { x: 600, y: 450 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
                base: 16, altitude: 32, color: 0xff0000
            },
            prefab: { 
                type: PsyanimEvadeAgentPrefab,
                params: {
                    maxSpeed: 3,
                }
            },
            components: [
                {
                    type: PsyanimEvadeAgent,
                    params: {
                        target: {
                            entityName: 'player'
                        }
                    }
                }
            ]
        }
```

By this point, your `InteractiveEvadeAgent.js` file should look like [this](https://github.com/thefinnlab/psyanim-overview-tutorials/blob/master/src/InteractiveEvadeAgent.js).

Reload your browser to see this in action!

This wraps up our discussion of the `trial end conditions` in the `PsyanimJsPsychPlugin`.

However, note that by writing custom `PsyanimComponents`, it is possible to create other custom `trial end conditions`.  The possibilities in are endless!