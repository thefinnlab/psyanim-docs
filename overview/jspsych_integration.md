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