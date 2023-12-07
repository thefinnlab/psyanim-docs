# <ins>Psyanim 2.0: jsPsych Integration Tutorial</ins>

***Note: Full tutorial project + source code can be found [here](https://github.com/thefinnlab/hello-psyanim2) in the `psyanim-jspsych-integration` branch.***

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

You can check out the [database schema](/overview/database_schema.md) to see more details on the structure and relationships of each of these collections.

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

## 9. Visualizing trial data with Experiment Viewer

To visualize data recorded from completed experiments, `psyanim2` ships with a tool called `Psyanim Experiment Viewer`.

Once an experiment's data has been saved to `firebase firestore`, the entire experiment can be played back using the `trial-metadata` for each trial and the associated trajectories store in the `/animation-clips` collection.

To get started, clone the [Psyanim Experiment Viewer](https://github.com/thefinnlab/psyanim-experiment-viewer) repository.

Next, we need to add a `private key file` for our `firebase project` to our `Experiment Viewer` root folder so the app is able to access our `firestore` data. 

---

Similar to how `jsPsych experiments` using the `PsyanimJsPsychPlugin` require a `firebase.config.json` in order to authenticate with a `firebase firestore` cloud service, the `Experiment Viewer` requires a `private key file` for your `firebase` service account.

To generate this `private key file`, follow the instructions [here](https://firebase.google.com/docs/admin/setup#initialize_the_sdk_in_non-google_environments).

Once you have downloaded your `private key file`, move it to a secure location in your local development environment for future use.

Copy this `private key file` into the root of the `Psyanim Experiment Viewer` project, renaming the copy to `'service-account.json'` and you're all setup to access your `firestore` data.

---

Now that we've got our `service-account.json` file in the root of our `Experiment Viewer` project, we're ready to build the app and load it in the web browser.

First let's install dependencies with:

```bash
npm i
```

Next, let's bundle the client app with:

```bash
npm run build
```

Then, let's start the `Psyanim Experiment Viewer` server to host the app:

```bash
npm run serve
```

Finally, in a browser, navigate to `localhost:7000` and you should see the experiment viewer app!

---

When you first load experiment viewer up, you will see a trial played back.  This trial is part of a collection of trials you can view.

To navigate through the available trials to view, use the `J` and `K` (for 'previous' and 'next' trials, respectively) keys on your keyboard.

To replay the current trial from the beginning, press the `Spacebar`.

Great work - you've mastered the basics of using `Psyanim Experiment Viewer`.

Don't worry abourt the `Trial Collection File Name` input field or the `Save Trial ID` button for now.  We will discuss the use of that later.

## 10. Custom queries with psyanim-cli

By default, `Psyanim Experiment Viewer` will load up every single trial that's ever been recorded in the `firestore` database associated with your `service-account.json` key file.

This is not ideal, however, as the number of trials in the database could be enormous, and ultimately cause `Experiment Viewer` to crash.

To avoid loading the entire database into the `Experiment Viewer`'s server memory, we should write a custom query script to filter out the data we want by some constraint(s).

However, before we look at how to apply custom queries to filter the data we want from `firestore`, it would be nice if we had some easy way to do quick exploratory analysis on what's currently in the database.

For this, `psyanim-cli` is just the tool for the job, as it has a feature to run custom query scripts at any time against the `firestore` database associated with 

--- 

In the root of your `Experiment Viewer` project, run the following command:

```bash
psyanim --query ./sample_firestore_queries/listCollections.js
```

You should've seen the following output:

```
Running query...

Found collection with id:  animation-clips
Found collection with id:  jspsych-experiment-data
Found collection with id:  session-logs
Found collection with id:  trial-metadata
```

So what happened here?  To illuminate this a bit more, open up `./sample_firestore_queries/listCollections.js` and you should see the following code:

```js
export function psyanimCliQuery(db) {

    db.listCollections()
        .then((collections) => {

            collections.forEach(c => {
                console.log('Found collection with id: ', c.id);
            });
        });
};
```

The javascript module `listCollections.js` is a `custom query script` that exports a single function: `psyanimCliQuery(db) { ... }`.

`psyanim-cli` can be used to run these `custom query scripts` from any directory that has a `service-account.json` file in it, simply by passing `--query` or `-q` and then the path to the query script module.

The only constraints on the query script is that it must export a function with the following signature: `psyanimCliQuery(db) { ... }`.

The `psyanimCliQuery(db)` function will be called by psyanim-cli and passed an instance to the `cloud firestore service` object from the [firebase admin sdk](https://firebase.google.com/docs/firestore/quickstart#initialize).

With this `cloud firestore service` object reference inside the `psyanimCliQuery` function, you can provide any custom query on the database you want, and can also run any other javascript code on that data that you want, including writing it out to disk or reading from disk and writing it out to the cloud database!

The `listCollections.js` sample query script is just a very simple example of how to query the `cloud firestore service` object to get all the names of the collections in the firestore database, and then print them to the console.

Some more complex, but very useful, sample scripts in the `./sample_firestore_queries/*` directory are the `dumpDatabaseToDisk.js` script and the `restoreCloudDB.js` script.  They allow you to pull the entire cloud db locally and write it out to disk as flat-files with the same structure as the collections in the cloud db, and vice-versa (reading a local flat-file db into the cloud service).

---

Now that we're a bit familiar with custom queries, let's see how we casn perform one of these custom queries as a filter for the data presented by `Psyanim Experiment Viewer`.

Let's inspect the contents of `./sample_firestore_queries/defaultDataProvider.js`:

```js
/**
 * Data provider should query the 'firebaseClient' (can do a custom query)
 * and then pass the array of docs to the 'dataHandler'.
 * 
 * @param {*} firebaseClient 
 * @param {*} dataHandler 
 */
export function dataProvider(firebaseClient, dataHandler) {

    /**
     *  firebaseClient.db is a firestore db reference.
     *  
     *  To see how to perform custom queries:
     *  https://firebase.google.com/docs/firestore/query-data/queries
     */
    firebaseClient.db
        .collection('trial-metadata')
        // .where('data.experimentName', '==', 'Second Experiment')
        // .where('data.userID', '==', 'UserC')
        // .where('data.sessionID', '==', '94f95b37-ea61-49ec-a90d-2dd21d4c6409')
        .get()
        .then((querySnapshot) => {

            let docs = querySnapshot.docs;

            // can also filter the returned query result for more complex queries
            // let docs = querySnapshot.docs.filter(doc => {
            //     return doc.get('data.userID') == 'UserC';
            // });

            dataHandler(docs);
        });
};
```

Similar to the custom query scripts accepted by `psyanim-cli`, the `Psyanim Experiment Viewer` also accepts custom query scripts as its input.

When `Psyanim Experiment Viewer`'s server starts up, it registers a callback with a `dataProvider` function when it executes it, allowing the data provider to fetch data and pass it to the callback for the `Experiment Viewer` to process.

So, `Experiment Viewer` is not aware of where its data actually comes from - so long as the data is provided to the callback it registered.

The signature of this function is: `dataProvider(firebaseClient, dataHandler)`, where `firebaseClient` is the `cloud firestore service` object from the `firebase admin sdk` (just like `psyanim-cli`), and the `dataHandler` is a callback that will send data to the `Experiment Viewer`.

By default, if no custom `dataProvider` module is supplied to `Psyanim Experiment Viewer`, it will use the `defaultDataProvider.js` under `./sample_firestore_queries`.

The `defaultDataProvider.js` just pulls all of the documents in the firestore `/trial-metadata` collection and passes them to the `Psyanim Experiment Viewer` for viewing.

Each `trial` in our database has a corresponding `firestore document` in the `/trial-metadata` collection, so you can see how large this collection might grow.

To see how to filter the data presented by `experiment viewer`, let's create a custom `dataProvider` module.

Let's create a new custom data provider mdoule using `psyanim-cli`:

```bash
psyanim --dataprovider myCustomDataProvider.js
```

Replace the body of the function with the following query:

```js
...

export function dataProvider(firebaseClient, dataHandler) {

    firebaseClient.db
        .collection('trial-metadata')
        .where('data.sceneKey', '==', 'WanderScene')
        .get()
        .then((querySnapshot) => {

            dataHandler(querySnapshot.docs);
        });
};

...
```

The only constraint here is that the exported `dataProvider` function signature must match what we had in the `defaultDataProvider.js` module.

Other than that, we can run whatever queries we want and do any custom filtering on the data as necessary.

Here, you'll see a `.where()` method in which we specify that we only want to see trials for which the `sceneKey` field is equal to `WanderScene`.

To run `Psyanim Experiment Viewer` with our new custom query module, the command line syntax is:

```bash
npm run serve ./src/myCustomDataProvider.js
```

After running the server with this new `dataProvider` module, reload the app in your browser and you should only see scenes where the `sceneKey` is 'Wander Scene`.

In general, the syntax for running `Psyanim Experiment Viewer` with a different `dataProvider` is:

```bash
npm run serve <path-to-data-provider>
```

Great work!  In summary, the experiment viewer can be run with any custom `dataProvider` module in order to limit or filter the data that it presents during usage in the browser.

## 11. Real-time playback of trial-collections during experiment

***WARNING: For this section, make sure the `firebase cloud project` that your `firebase.config.json` points to is not a production database, so we can *safely* overwrite its contents!***

Out-of-the-box, Psyanim 2 comes with `components` and `template scenes` that allow us to quickly setup `real-time playback` of any recorded `trial` in `firebase` during an experiment.

To see this in action, there is already a demo project in the `hello-psyanim2` repository under the branch `experiment-player-example`.

---

In our `hello-psyanim2` repo, let's checkout this branch with:

```bash
git checkout experiment-player-example
```

Your `firebase.config.json` should still be in the root of this project.

Delete the `./dist` and `./node_modules` directories, as these contain dependencies and packaged builds for a different branch.

Run `npm i` again to install the dependencies for this branch.

Run a watch service with `npm run watch` in one terminal, and the local server in another terminal with `npm run serve`.

Now this `experiment-player-demo` branch is all ready to use.  However, our database doesn't have the data it expects yet.

---

Let's update the `firestore cloud database` to have the proper data needed to run the example demo.

Switch back over to your `Psyanim Experiment Viewer` project, as it has some scripts to help us push some test data into the cloud database.

First off, let's delete everything in our `firestore cloud database` with the following command:

```bash
npm run delete-cloud-db
```

Next, let's open up `./sample_firestore_queries/restoreCloudDB.js` and replace the `localDataStoreRootPath` at the top of the file with:

```js
...
const localDataStoreRootPath = './sample_firestore_datasets/2-experiments-5-users';
...
```

This sample script shows how to restore a `firestore cloud datbase` from a local flat-file copy of a `Psyanim 2 Database`.

By changing the `localDataStoreRootPath` in this `restoreCloudDB.js` query script, we've told it to restore the `2-experiments-5-users` database.

This database has trials 2 different `experiments` with 5 different `users`, so it's a good dataset for learning & regression testing with.

You can create new local copies of your `firestore cloud database` anytime using the `dumpDatabaseToDisk.js` script.

For now, however, we will work with the `2-experiments-5-users` dataset for our demo.

To run this script, use `psyanim-cli`:

```bash
psyanim -q ./sample_firestore_queries/restoreCloudDB.js
```

You should now have all of the data from the `2-experiments-5-users` directory in your `firestore cloud database`.

---

Now that we have a test dataset to run our demo with, we can switch back over to our `hello-psyanim2` repo's `experiment-player-example` branch.

We've already started a watch service and local server on port 3000, so let's navigate to `localhost:3000` in our browser to see this in action.

You should see all the trials playing from `UserB`, as this is what the `experiment-player-example` was configured to do by default.

---

The way the `experiment-player-example` demo app knew to only pull data from `UserB` to playback during the experiment is because that's all that was added to it's `Trial Collection Files`.

A `Trial Collection File` in `Psyanim 2` is just a file that contains metadata about a collection of `trials` in our database.

In this case, we only added trials to this `Trial Collection File` to be played back that were from `UserB`.

To generate this `Trial Collection File`, it's convenient to use `Psyanim Experiment Viewer` because it allows you to visualize each trial before adding it to a file.

It's also possible generate this file by creating a custom query module to run in `psyanim-cli`, but that's a more advanced topic we won't approach here.

---

To get a feel for how to generate these `Trial Collection Files`, let's hop back over to our `Psyanim Experiment Viewer` project.

Let's create a new `Trial Collection File` that only contains `trial metadata` for `UserC` instead of `UserB`.

Create a new `data provider` module for `Experiment Viewer` with the following command:

```bash
psyanim -d myDataProviderUserC
```

Open this file up and replace the body of the `dataProvider()` function with:

```js
export function dataProvider(firebaseClient, dataHandler) {

    firebaseClient.db
        .collection('trial-metadata')
        .where('data.userID', '==', 'UserC')
        .get()
        .then((querySnapshot) => {

            let docs = querySnapshot.docs;

            dataHandler(docs);
        });
};
```

Now let's start the `Psyanim Experiment Viewer` with the following command (may need to kill the old instance if you didn't already):

```bash
npm run serve ./src/myDataProviderUserC.js
```

If you peek through all the trials presented by `Experiment Viewer`, you'll see there are only trials for `UserC` now.

Remember, this is thanks to the custom `data provider` module we passed to the server on startup.

If you inspect the page, there is an `input text field` with the label `Trial Collection File Name` and it should contain `defaultTrialCollection`.

This is the name of `Trial Collection File` we're going to be adding trials to.  The default name is fine for our purposes here, but you can always change it.

You'll also notice there is a `Save Trial ID` button.  If you click this on any trial, you'll notice a `./trial_collections` directory was created in your `Experiment Viewer` project with a `defaultTrialCollection.json` file in it.

You have just added this `trial` to your `Trial Collection File`.  The `Save Trial ID` button will add any trial to the `Trial Collection File` only if it doesn't exist already.

Go ahead and run through all the trials from `UserC`, saving them all to your `defaultTrialCollection.js` file.

---

Now that we've created a new `Trial Collection File` containing all of the `trials` for `UserC`, we can simply copy this file from our `Experiment Viewer` project into the `./src` directory of our `hello-psyanim2` repo, replacing the previous `defaultTrialCollection.json` file that was there.

Since we already started the watch service and local server for this project, you should be able to just reload `localhost:3000` in your browser and see the new data from `UserC` playing back in real-time from `firestore`.

Great work - you've created your first `Trial Collection File` with hand-selected trials and added them to an experiment to play back in real-time!

---

To wrap things up, let's break down how the real-time playback was setup in the `index.js` of the `experiment-player-example` branch of our `hello-psyanim2` repo.

If you peek at the `psyanim2` package imports at the top of the `index.js`, notice the trial loaders and scene templates at the end of the list.

The `PsyanimJsPsychTrialLoader` is a `Psyanim Component` that loads the data for our experiment from `firestore` on startup.

The `PsyanimJsPsychTrialSelector` is a `Psyanim Component` that randomly samples trials (without replacement) to be played back one at a time during the experiment.

The `PsyanimJsPsychExperimentLoadingSceneTemplate` is a template scene we can use to load all of the trials that will be used in our experiment.

The `PsyanimJsPsychExperimentPlayerSceneTemplate` is a template scene we can use to playback any trial in our experiment.

The template scenes are there to minimize the boiler-plate you have to write, since all loading & playback scenes will look very similar, if not the same.

Feel free to craft these by hand if you want to understand things better or customize them though!

Everything else in the `index.js` should be familiar to you, so we can skip ahead to where we setup the `experiment loader scene trial` and the `main playback scene trials`.

The `experiment loader scene trial` has to run *before* any `trials` can be played back from `firestore`.

It only has 2 parameters you need to configure on the `PsyanimJsPsychTrialLoader` component - `trialInfo` and `documentReader`.

The `trialInfo` is just an array of trial objects from your `Trial Collection Files`.  The `documentReader` is just the `firebase client` reference.

The `main playback scene trials` are the trials that will actually play back data from `firestore`.

This scene only has 1 parameter on a `PsyanimJsPsychTrialSelector` component that you need to configure - `trialInfo`.

The `trialInfo` here is also just an array of objects from your `Trial Collection Files`.

---

And this wraps up the `Psyanim-jsPsych Integration` tutorial!

Great work!  There's still a lot more to learn and many more features we couldn't discuss here.

A great way to keep exploring is to create new experiments, scenes and components, and dive into some of the `psyanim2 core` code itself.