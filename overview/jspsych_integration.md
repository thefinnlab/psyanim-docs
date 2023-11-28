# Psyanim 2.0: jsPsych Integration Tutorial

***Note: Full tutorial project + source code can be found [here](https://TODO-brokenlink).***

***If, at any time, you're uncertain of what the final code should look like for a section, please refer to the project source code above.***

## 1. Psyanim-jsPsych Integration

`Psyanim 2.0` is designed as a standalone [Phaser](https://phaser.io/) app which renders to an `HTML Canvas`, as any other `Phaser` app does.

In order to use `Psyanim 2.0` in [jsPsych](https://www.jspsych.org/), a `jsPsych plugin` was developed: `PsyanimJsPsychPlugin`.

In this tutorial, we'll walk through how to use a `Psyanim scene` as a `jsPsych trial`, as well as how to create variations of the same scene in different trials, each with their own independent sets of parameters for the same scene.

## 2. The Anatomy of the `PsyanimJsPsychPlugin`

The main learning curve in `Psyanim 2.0` is understanding how to build interactive content in the `Psyanim scene definitions` and extending the framework with custom `Psyanim components`.

The `PsyanimJsPsychPlugin` is designed to allow you to easily run `Psyanim Scenes` as `jsPsych trials`.

The plugin also handles any persistence in firebase for you, including custom parameters, and this is all configurable per trial via the `PsyanimJsPsychTrial` APIs.

---

Let's take a moment to walk through some of the fundamentals of using the `PsyanimJsPsychPlugin` in a `jsPsych experiment`.

All of the `jsPsych` usage is the same as any other experiment, as described in the [docs](https://www.jspsych.org/).

Before using the `PsyanimJsPsychPlugin` in any `jsPsych experiment`, you must first start the `PsyanimApp` by calling `PsyanimApp.instance.run()`.

Note that we set the `PsyanimApp canvas` to be invisible at the start of the experiment.  This is desirable in most cases, as we often won't be using the `PsyanimJsPsychPlugin` for the first jsPsych trial in our timeline.

The `PsyanimJsPsychPlugin` must then be initialized with the following:

- A `user ID`
- An `experiment name`
- (Optionally) A `document writer`

The `user ID` and `experiment name` can come from any source, e.g. the browser or an external web service.

The `document writer` is an interface to write out specific types of JSON documents to a persistent data store.  This can be any underlying data store, so long as the `document writer` for it implements the interface properly.

By default, `Psyanim 2` ships with tight integration with `Google Firestore`, which is part of their `Firebase` cloud services.  For experiments using `Firestore` as the backend database, the `PsyanimFirebaseBrowserClient` will serve as the `document writer` for the `PsyanimJsPsychPlugin`.

Once this information has been configured in the `PsyanimJsPsychPlugin`, all that's left to do is setup our `jsPsych trials` in a `timeline` and run `jsPsych`.

For other `jsPsych plugins`, refer to the documentation for that plugin to see how to configure the `trial object`.

For the `PsyanimJsPsychPlugin`, the `PsyanimJsPsychTrial` class should be used to generate the jsPsych `trial object` for the `timeline`.

Once a `PsyanimJsPsychTrial` instance has been created and configured as desired, it's `jsPsychTrialDefinition` member can be added directly to the jsPsych `timeline` at any point during the timeline setup.

---

And that's all there really is to the `PsyanimJsPsychPlugin` interface!

It's not much more complex than using any other `jsPsych plugin`.

The complexity really comes from designing your `Psyanim Scenes` and implementing `Psyanim Components` as necessary.

## 3. Setting up our test project

***Pre-requisites:  All the pre-requisites in the [Hello Psyanim 2.0 tutorial](/overview/hello_psyanim_2.md#2-creating-a-new-psyanim-2-project) must be met.***

***You'll also need to setup a Firebase service account and project by following these instructions: [Get started with Cloud Firestore](https://firebase.google.com/docs/firestore/quickstart)***

To start, create a new empty Psyanim 2 project, following the same steps as the [Hello Psyanim 2.0 tutorial](/overview/hello_psyanim_2.md#2-creating-a-new-psyanim-2-project).

Using `psyanim-cli`, let's create a new `scene` called `WanderScene` and a new `component` called `MyCustomDebugger`:

```bash
psyanim --scene WanderScene
psyanim --component MyCustomDebugger
```

---

Remember to open a couple of terminals and startup a watch service in the first one with: `npm run watch`

Then, in the second terminal, start the local server for testing your client app using: `npm run serve`

---

The last thing you'll need is a `Firebase config object` serialized into JSON in a file in the root of your project directory named `firebase.config.json`.

To get this object for your app, follow these instructions: [Get a config object for your web app](https://support.google.com/firebase/answer/7015592?hl=en#web&zippy=%2Cin-this-article).

You can copy the config object into your `firebase.config.json` file and manually serialize it to JSON just by wrapping the object keys in double quotes.

When you are done, the contents of your `firebase.config.json` will have the following structure, with same keys, but different values:

```js
{
  "apiKey": "AIzaSyDsf7IhZP9QIjGRZgYTNV0vAQEy1vK3Pgo",
  "authDomain": "psyanim2-backend.firebaseapp.com",
  "projectId": "psyanim2-backend",
  "storageBucket": "psyanim2-backend.appspot.com",
  "messagingSenderId": "493573326180",
  "appId": "1:493573326180:web:33ba5170ccd14f9aefe671",
  "measurementId": "G-DVJX659VWL"
}
```

Now we have all the key pieces of our project in place.  Next, we'll start implementing our experiment!

## 4. Implementing our Wander Scene

In this experiment, we'll be creating multiple variations of a `wander scene`, or a scene containing agents that will randomly wander around.

To do this, we'll create a single `scene definition` which will serve as a template for creating numerous trials as variations of this scene.

Open up `WanderScene.js` and update the `psyanim2` import at the top to include the `PsyanimWanderAgentPrefab`.

Then, add the following entity definition to the `entities` array of the `scene definition`:

```js
{
    name: 'agent',
    initialPosition: 'random',
    shapeParams: {
        textureKey: 'wanderTestTexture',
        shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
        base: 16, altitude: 32, 
        color: 0xffc0cb            
    },
    prefab: {
        type: PsyanimWanderAgentPrefab,
        params: {
            debug: false
        }
    }
},
```

Next, open up your `index.js`, remove the `EmptyScene.js` references (can delete that file too), import the `WanderScene.js` as `WanderScene`, and create a new trial in our timeline:

```js
// setup a single wander scene trial
let wanderSceneTrial = new PsyanimJsPsychTrial(WanderScene, WanderScene.key);

timeline.push(wanderSceneTrial.jsPsychTrialDefinition);
```

Reload the experiment in the browser and you should see a single triangle-shaped agent wander around in your first `WanderScene` trial!

## 5. Implementing a custom debugger

`psyanim2` has an abstraction layer called `PsyanimDebug` which is designed to allow log statements to be redirected to any output source (browser console, nodejs console, flat file, cloud database, or any combination of these) without any code changes to `Psyanim Components`.

Let's give this feature a spin by implementing a custom debug component which logs data out to `firebase firestore` using the `PsyanimDebug.log()` APIs.

Open up `MyCustomDebugger.js` and update the `psyanim2` import to also import the `PsyanimWanderBehavior` component.

Then, add another import for the `PsyanimDebug` class from the `psyanim-utils` package.

---

Let's review the strategy for this custom debug `component` implementation, real quick.  Remember that this is meant to be a contrived example, so not super useful here, but hopefully we'll see how it could be used in a more useful manner.

The `MyCustomDebugger` component will be added to an entity in each `wander scene trial`.

After the `scene` is `created` (Phaser's terminology for each time a scene is initialized), the `MyCustomDebugger` component will search for the first `PsyanimWanderBehavior` component it can find attached to an `entity` in the scene.

Then, at a frequency of `1 hz`, the `MyCustomDebugger` component will record the private `_angle` property of the `PsyanimWanderBehavior` using the `PsyanimDebug.log()` APIs.

For now, this will only send the output to the browser's console.  

In a later section of this tutorial, we'll send that output to a document in `firebase firestore` by simply configuring a different logger type, but without modifying our `MyCustomDebugger` component code.

---

Let's roll up our sleeves and code this `MyCustomDebugger` component now!

Add a public property to our component called `loggingInterval`.

Then, update the constructor to initialize this property, as well as creating a private field called `_timer`:

```js
constructor(entity) {

    super(entity);

    this.loggingInterval = 1000;

    this._timer = 0;
}
```

Next, in the `afterCreate()` method of our `component`, we'll update it to find the first `entity` in the scene with a `PsyanimWanderBehavior` component attached to it and output a debug log statement:

```js
afterCreate() {

    super.afterCreate();

    // save off the wander behavior of the first agent we find that has one
    this._wanderBehavior = this.scene.getComponentsByType(PsyanimWanderBehavior)[0];

    PsyanimDebug.log('New wander scene created, logging state for agent: ', this._wanderBehavior.entity.name);
}
```

Now let's create a `_logWanderBehaviorState()` method on our component that simply logs any state we're interested in:

```js
_logWanderBehaviorState() {

    PsyanimDebug.log('_angle = ', this._wanderBehavior._angle);
}
```

Let's finish up this component's implementation by updating our `_timer` each frame, and then calling `_logWanderBehaviorState()` every `loggingInterval`.  Modify your `update(t, dt)` method to do the following:

```js
update(t, dt) {
    
    super.update(t, dt);

    this._timer += dt;

    if (this._timer > this.loggingInterval)
    {
        this._logWanderBehaviorState();

        this._timer = 0;
    }
}
```

Back in your `scene definition` in `WanderScene.js`, import the `MyCustomDebugger.js` component as `MyCustomDebugger` and add a new `entity` with this component attached:

```js
{
    name: 'myDebugger',
    components: [
        { type: MyCustomDebugger }
    ]
}
```

Reload your experiment in the browser and you should see our `'New wander scene created, logging state for agent: '` message and the `PsyanimWanderBehavior::_angle` property getting printed every second while the trial is running!

## 6. Creating trial variations with different parameters

Let's create a few additional `wander scene` trials with different parameters to add to our timeline.

In each trial, we will vary the `duration` of the trial, the number of `wander agents` in the scene, and the `agent colors`.

We will also vary some of the parameters of the `wander prefab`.

To start, let's define some trial parameters for each trial in our `index.js`:

```js
...

/**
 *  Setup multiple wander scene trials each with different parameters & instanced entities
 */
let nTrials = 3;

let trialDurations = [5000, 10000, -1];
let trialAgentInstances = [3, 6, 20];
let trialAgentColors = [ 0x32CD32, 0xBC13FE, 0xFFC0CB ];

let trialPrefabParams = [
    {
        radius: 30,
        maxWanderSpeed: 3.0,
        maxAcceleration: 0.02,
        maxAngleChangePerFrame: 10
    },
    {
        radius: 30,
        maxWanderSpeed: 1.5,
        maxAcceleration: 0.02,
        maxAngleChangePerFrame: 10
    },
    {
        radius: 50,
        maxWanderSpeed: 3,
        maxAcceleration: 0.2,
        maxAngleChangePerFrame: 20
    }
];

...
```

There are many ways we can manage our parameter-sets for each trial in our experiments, and this is just a simple one that's (hopefully) easy to understand quickly.

That said, there are no hard-and-fast rules for how we manage our parameter-sets, so please feel free to modify this approach to suit your own experiment's needs.

Ultimately, the only thing that matters is that the `PsyanimJsPsychTrial` object you add to your timeline has all the information it needs about your scene and parameters for each trial.

How you get that information into it is intentionally left up to the user.

Here, we're setting up the parameters for 3 trials, thus we see an `array` of length `3` for each `parameter type`.

The parameter-set for each trial is identified by an index into each array.

So, for example, the 2nd trial of this set would be index `0` in these parameter arrays, giving it a trial duration of `10000`, `6` agent instances, and an agent color of `0xBC13FE`.

The second trial would also have a `prefab parameter-set` of: 

```js
{
    radius: 30,
    maxWanderSpeed: 1.5,
    maxAcceleration: 0.02,
    maxAngleChangePerFrame: 10
},
```

Now that we've defined our parameter-sets for each trial, let's loop over these trial parameter-sets and add a trial to our timeline for each one as follows:

```js
...

for (let i = 0; i < nTrials; ++i)
{
    // create trial from template using a unique scene key
    let wanderSceneKey = WanderScene.key + '_' + i;
    wanderSceneTrial = new PsyanimJsPsychTrial(WanderScene, wanderSceneKey);

    // set trial params
    wanderSceneTrial.duration = trialDurations[i];

    // set entity params
    wanderSceneTrial.setEntityParameter('agent', 'instances', trialAgentInstances[i]);
    wanderSceneTrial.setEntityShapeParameter('agent', 'color', trialAgentColors[i]);

    // set prefab params
    let currentPrefabParams = trialPrefabParams[i];

    for (let paramKey in currentPrefabParams)
    {
        wanderSceneTrial.setPrefabParameter('agent', paramKey, currentPrefabParams[paramKey]);
    }

    // setup recording params
    wanderSceneTrial.addAgentNamesToRecord(['agent*']);

    // push trial into timeline
    timeline.push(wanderSceneTrial.jsPsychTrialDefinition);
}

...
```

Here, we loop over each `trial parameter-set` we've create, instantiating a new `PsyanimJsPsychTrial` for each one, using the `WanderScene` definition.

Before pushing the newly created trials into our `jsPsych timeline` array, we set the `trial params`, `entity params`, and `prefab params` using the previously declared parameter arrays and the appropriate `array index` for that `trial`.

Great work - reload your experiment in the browser and you should see 3 new trials, each with the corresponding number of agents, colors, and trial durations.

## 7. Saving experiment data out to firebase firestore

The `PsyanimJsPsychPlugin` is designed to handle all interactions & writing of data to firebase firestore during the experiment w/ minimal input from the experiment developer.

There are various types of data that can be persisted in firebase, each with its own `firestore collection` type:

- `trial-metadata`: Metadata about each trial, including trial parameters.
- `animation-clips`: These are agent/entity trajectories recorded during a trial.
- `jspsych-experiment-data`: This is the `jsPsych.data` object created during the trial.
- `session-logs`: This is the output of any `PsyanimDebug.log()`, `PsyanimDebug.warn()` or `PsyanimDebug.error()` statements called during the experiment.
- `state-logs`: This is the state information (states/transitions + times) for any agent running a particular state-machine for a behavior, within the `Psyanim Decision-Making Framework`

You can check out the [database schema](TODO://broken-link) to see more details on the structure and relationships of each of these collections.

For now, just know that these exist and the data recorded in each of them is highly configurable per-trial via the and `PsyanimJsPsychTrial` interfaces.

To enable data-saving to `firebase firestore` in your experiment, you first need to import your `firebase.config.json` in your `index.js`, which should be at the root of your project:

```js
...
import firebaseJsonConfig from '../firebase.config.json';
...
```

Then, in the section of the `index.js` where we setup the `PsyanimJsPsychPlugin`, add the following code to create a `PsyanimFirebaseBrowserClient` and set it as the `document writer` of the `PsyanimJsPsychPlugin`:

```js
...
const firebaseClient = new PsyanimFirebaseBrowserClient(firebaseJsonConfig);
PsyanimJsPsychPlugin.setDocumentWriter(firebaseClient);
...
```

And that's all!  The `PsyanimJsPsychPlugin` will now save all of it's data out to `firestore` at the end of every trial.

## 8. Writing jsPsych.data out to firebase firestore

As with any `jsPsych` experiment, we may want to capture the contents of the `jsPsych.data` object at various points throughout the experiment.

In most cases, we do not want to wait until the end of the experiment to do so, as a program crash or a closed browser tab could cause us to lose data for trials that have already been completed.

Instead, we will use a custom [jsPsych extension](https://www.jspsych.org/7.3/overview/extensions/) written specifically for `psyanim 2` called the `PsyanimJsPsychDataWriterExtension`.

With this extension enabled, and as long as your experiment's `firebase client` and `PsyanimJsPsychPlugin` is all configured properly, the `jsPsych.data` object will get written out at the end of any trial for which the extension is added.

The first thing you should do to use this extension is to update your `psyanim2` import statement to include the `PsyanimJsPsychDataWriterExtension` class.

Then, we need to update our `initJsPsych()` call to enable the use of the extension and supply the extension the necessary experiment-level parameters:

```js
...

/**
 *  Setup jsPsych experiment
 */
const jsPsych = initJsPsych({

    extensions: [
        { 
            type: PsyanimJsPsychDataWriterExtension, 
            params: { 
                documentWriter: firebaseClient,
                userID: userID,
                experimentName: experimentName
            }
        }
    ],
});

...
```

For non-PsyanimJsPsychPlugin trials, you will just manually add an `extensions` field to your `trial object` containing a reference to this extension.

As an example, this is how you'd add this extension to the `'welcome' trial` in our experiment:

```js
// 'Welcome' trial
timeline.push({
    type: htmlKeyboardResponse,
    stimulus: "<p style='text-align:center'>Welcome to the experiment.  Press any key to begin.</p>",
    extensions: [
        { type: PsyanimJsPsychDataWriterExtension }
    ],
});
```

For all PsyanimJsPsychPlugin trials, you can simply call `addExtension` on the `PsyanimJsPsychTrial` object before pushing it onto the `jsPsych timeline`:

```js
...

// add jsPsych.data writer extension
wanderSceneTrial.addExtension(PsyanimJsPsychDataWriterExtension);

...
```

Go ahead and add the extension to every trial in your timeline, including the `'end' trial`.

And voila, that's all you need to do to save out the `jsPsych.data` object at the end of any trial.

The data will be saved under the `/jspsych-experiment-data` collection in `firestore` and the `document ID` will be the `session ID` for that experiment session.

## 9. Visualizing experiment data with experiment viewer

***// TODO: explain the pre-requisite for this section is to download the `service-account.json` file and how to do this***

To visualize data recorded from completed experiments, `psyanim2` ships with a tool called `Psyanim Experiment Viewer`.

Once an experiment's data has been saved to `firebase firestore`, the entire experiment can be played back using the `trial-metadata` for each trial and the associated trajectories store in the `/animation-clips` collection.

// TODO: need to look at `react-mvc-tests` branch of experiment viewer repo b.c. it looks like that is where you added the modular query filters via cmd line syntax
//          initial testing looks like it's in working order... but do a bit more testing to be sure before integrating it into `dev` and `master`

// TODO: elaborate on these steps here

- checkout experiment viewer app from: https://github.com/thefinnlab/psyanim-experiment-viewer
- add service-account.json to root of app directory
- npm i
- npm run build (to build client)
- npm run serve (to run server)

## 10. Custom queries with psyanim-cli

- show how the `psyanim-cli` can be used for exploratory analysis and building query filters for experiment viewerf
- show how the `psyanim-cli` can be used to build little cmd line tools around firebase queries

## 11. Saving trial collection files from experiment viewer

## 12. Real-time playback of trial-collections during experiment

