# <ins>Interactive Predator Chasing</ins>

***Pre-requisites: You'll want to make sure you complete the [Firebase Setup tutorial](/overview/firebase_setup.md) because you'll need it to complete this tutorial!***

## 1. Overview and Project Setup

<p align="center" style="font-size: 12px;">
    <video width="640" height="360" controls>
    <source src="./videos/interactive-chase-section1.mp4" type="video/mp4">
    </video>
</p>

In this tutorial, we'll setup a new experiment with multiple interactive trials and see how to record each trial's metadata, as well as agent and player trajectories to `Google Firebase`.

The trials will involve a `predator agent` that chases the `mouse cursor` with a certain degree of `subtlety`.

The `subtlety` is controlled by a maximum angle which the predator can deviate from a straight-line path to the prey.

At a fixed frequency, the predator's trajectory during a chase will be randomly picked such that it stays within this maximum angle of deviation from the path to the prey.

The greater the `subtlety`, the more the predator will tend to stray from directly chasing the mouse cursor.

The lesser the `subtlety`, the more the predator will tend to move straight towards the prey during a chase.

---

For this tutorial, let's create a new project with `psyanim-cli`.

Create a new project directory called `interactivePredatorChase` and navigate to it in a terminal.

Run the following command to create a new experiment project with psyanim-cli:

```bash
psyanim init
```

Next, let's add a `template scene` that's already setup up for a predator agent to chase our mouse cursor.

To see what template scenes are available, run the following command to display the psyanim-cli `template` command's help menu:

```bash
psyanim template --help
```

The valid subcommands listed are all template scenes we can add to our project.

Let's go ahead and add the `predatormousev2` to our scene with the following command:

```bash
psyanim template:predatormousev2 -o ./src/scenes/
```

---

Finally, all we need to do is add this to our `jsPsych experiment timeline` in `index.js`.

First let's delete `EmptyScene.js` under `./src`, as we won't need that blank scene.

Open up `index.js` under the `/src` directory and delete the `EmptyScene` import statement at the top.

Add the following import statement in its place:

```js
import InteractivePredatorMouseV2 from './scenes/InteractivePredatorMouseV2.js';
```

Next, scroll down to the place in code where we added that `EmptyScene` to a `PsyanimJsPsychTrial` object and delete the following lines:

```js
// psyanim empty scene trial
let emptySceneTrial = new PsyanimJsPsychTrial(EmptyScene, EmptyScene.key);
// emptySceneTrial.saveTrialMetadata = true;
// emptySceneTrial.addExtension(PsyanimJsPsychDataWriterExtension);

timeline.push(emptySceneTrial.jsPsychTrialDefinition);
```

In that same location (between the 'Welcome Trial' and 'End Trial'), let's create a trial for our `InteractivePredatorMouseV2` scene and add it to our timeline with the following code:

```js
// first predator trial
let interactivePredatorMouseV2Trial = new PsyanimJsPsychTrial(
    InteractivePredatorMouseV2, InteractivePredatorMouseV2.key);

timeline.push(interactivePredatorMouseV2Trial.jsPsychTrialDefinition);
```

Make sure you have a watch service and dev server running, reload the experiment in your browser and you should see the green predator agent wandering about, occasionally turning red and charging after the mouse cursor before returning to a wander.

Great work -  you've setup an interactive predator-chase experiment!

## 2. Creating Trial Variations

<p align="center" style="font-size: 12px;">
    <video width="640" height="360" controls>
    <source src="./videos/interactive-chase-section2.mp4" type="video/mp4">
    </video>
</p>

So far, we've seen how we can create `jsPsych Trials` from `Psyanim Scene Definitions`.

In this section, we'll create multiple `jsPsych Trials` that are variations of the same `Psyanim Scene Definition.`

This is useful for experiments where we would like to see how a subject's perception of agent interactions changes with respect to a particular variable.

To start, let's update our `psyanim2 import statement` at the top of our `index.js` so it has the following imports:

```js
import { 
    PsyanimApp, 
    PsyanimJsPsychPlugin,
    PsyanimJsPsychTrial,

    PsyanimFirebaseBrowserClient,

    PsyanimConstants,

    PsyanimPredatorFSM

} from 'psyanim2';
```

---

Let's create 2 more `predator-chase` trials where the predator chases the mouse, and we'll do so by creating a new `PsyanimJsPsychTrial` object using the same scene definition as before, but with a different `scene key`.

Add the following code after the line where we push the first predator trial into the `jsPsych timeline` object:

```js
// second predator trial
let interactivePredatorMouseV2Trial2 = new PsyanimJsPsychTrial(InteractivePredatorMouseV2, InteractivePredatorMouseV2.key + '_2');
interactivePredatorMouseV2Trial2.setEntityShapeParameter('predator', 'shapeType', PsyanimConstants.SHAPE_TYPE.CIRCLE);
interactivePredatorMouseV2Trial2.setEntityShapeParameter('predator', 'radius', 12);

timeline.push(interactivePredatorMouseV2Trial2.jsPsychTrialDefinition);
```

Notice that we modified the second parameter to the `PsyanimJsPsychTrial` constructor as it passes the original scene's key, but with a '2' appended to it.

This is because the first argument to the `PsyanimJsPsychTrial` constructor is the scene definition object for the trial, while the second argument is a `unique key` associated with that scene and it's parameter-set.

So long as that `scene key` is different for every trial object we push into our timeline, we can reuse the same `scene definition` for as many trial objects as we'd like, allowing us to create multiple variations of the same scene.

To modify the parameters of the scene definition for the second trial, the trial object exposes a set of API methods that allow us to modify them and specify whether or not we want them to be recorded (we will discuss parameter recording in another tutorial).

For this trial, we use the `setEntityShapeParameter()` method, which allows us to modify any of the `shape parameters` of any `entity` in our scene definition.

We'll also change the `shapeType` of our `predator` entity to be a circle, and specify it's `radius` to be 12px.

---

Let's create one more `predator-chase` trial by adding the following code after the second trial is pushed into the jsPsych timeline:

```js
// third predator trial
let interactivePredatorMouseV2Trial3 = new PsyanimJsPsychTrial(InteractivePredatorMouseV2, InteractivePredatorMouseV2.key + '_3');

interactivePredatorMouseV2Trial3.setComponentParameter('predator', PsyanimPredatorFSM, 'maxWanderSpeed', 5.0);
interactivePredatorMouseV2Trial3.setComponentParameter('predator', PsyanimPredatorFSM, 'maxChargeSpeed', 5.0);

timeline.push(interactivePredatorMouseV2Trial3.jsPsychTrialDefinition);
```

Here, we modify two component parameters on the `predator` entity's `PsyanimPredatorFSM` components, `maxWanderSpeed` and `maxChargeSpeed`, increasing them to `5.0`, so we should see the entity traveling faster than the ones in the previous two trials.

---

Reload your experiment in the browser and you should now see two more trials in your jsPsych experiment.

In the first trial and third trials, you should see a triangle-shaped predator agent.

In the second trial, you should see a circle-shaped agent.

The first two trials' parameters are the same, except for the shape of the agent.

The third trial has the same shape as the first, but the agent should be moving significantly faster.

---

Now let's create 3 more trials, varying the `chase subtlety` parameter for the predator agent in each one.

Add the following code right after the line where we pushed the third interactive predator trial into the jsPsych timeline array:

```js
// chase subtlety trials
let subtletyValues = [ 5, 75, 180];

for (let i = 0; i < subtletyValues.length; ++i)
{
    let predatorChaseSubtletyTrial = new PsyanimJsPsychTrial(
        InteractivePredatorMouseV2, 'sublety_' + (i + 1));

    predatorChaseSubtletyTrial.setComponentParameter('predator',
        PsyanimPredatorFSM, 'subtlety', subtletyValues[i]);

    timeline.push(predatorChaseSubtletyTrial.jsPsychTrialDefinition);
}
```

In the code above, we declare an array of subtlety values we'd like to create trials with, and then loop over each subtlety value and create a trial in the jsPsych timeline, setting the appropriate `subtlety` value of the `PsyanimPredatorFSM` component for each trial.

These subtlety angles are angles defined with respect to a line from the predator to it's target prey.

The predator will approach the prey during a 'charge' with some degree of subtlety, where the angle of approach is somewhere between 0 degrees (head-on) or some random value no larger than the `subtlety` parameter.

The subtlety values represent a maximum angle between the line of sight from predator-to-prey and the actual direction the predator will move in.

For the subtlety value of 5 degrees, we expect the predator will basically be moving straight toward the prey.

For a subtlety value of 75 degrees, the predator will move towards the prey, but not directly towards it and never in a direction away from it.

And for a subtlety value of 180 degrees, the predator can move towards or away from the prey.

---

Great work! Start a watch service in your terminal with `npm run watch` and a dev server in another terminal with `npm run serve`.

Reload your experiment in the browser and you should see there are now 6 trials.

Remember, by default, you can use the `Enter` key on your keyboard to progress to the next `jsPsych trial`.

The first 3 trials share a constant subtlety parameter value.

The last 3 trials are our `chase-subtlety` experiment trials, where the subtlety values are `[5, 75, 180]` in each one, respectively.

## 3. Saving Data To Firestore

<p align="center" style="font-size: 12px;">
    <video width="640" height="360" controls>
    <source src="./videos/interactive-chase-section3.mp4" type="video/mp4">
    </video>
</p>

To start saving data to a `Google Firebase Firestore` database, there are just a few quick setup steps.

> Note that, if this is your first time ever using the `PsyanimJsPsychPlugin` to record data to firebase (i.e. your database is freshly setup and never used before), you'll also need to update the database security rules.  We will see how to do this later in this section.

At a high-level, the only thing that needs to be done to enable saving `trial metadata` out to firebase is for the `PsyanimJsPsychPlugin`'s `documentWriter` property to be set to a valid document writer.

For firebase, the document writer we'll want to use is the `PsyanimFirebaseBrowserClient`.

For the `PsyanimFirebaseBrowserClient` to communicate with firebase, you'll need to pass it a valid `firebase config` object.

So, to start, copy the `firebase.config.json` you created in the [Firebase Setup tutorial](/overview/firebase_setup.md) into the root of this project.

Next, let's open our `index.js` and uncomment the following `import` statement near the top of the file:

```js
import firebaseJsonConfig from '../firebase.config.json';
```

Then, under the section with the code comment `Setup PsyanimJsPsychPlugin`, uncomment the following two lines that create a `PsyanimFirebaseBrowserClient` and assign it as the `document writer` of the PsyanimJsPsychPlugin:

```js
const firebaseClient = new PsyanimFirebaseBrowserClient(firebaseJsonConfig);
PsyanimJsPsychPlugin.setDocumentWriter(firebaseClient);
```

Finally, you'll want to copy the "projectId" value from your `firebase.config.json` to your `.firebaserc` file's "projects.default" field value, so it matches the one in your `firebase.config.json`.

e.g. if the "projectId" from your `firebase.config.json` was "myTestProject", then your `.firebaserc` should look like the following:

```js
{
  "projects": {
    "default": "myTestProject"
  }
}
```

> If this is your first time using the `PsyanimJsPsychPlugin` to write data out to a freshly created Firestore database instance, you'll need to update the security rules of the database to allow writes to certain collections.
> To do this, simply run the following command in terminal:

```bash
npm run firebase-deploy-rules 
```

Now, if you navigate to your Firestore console in one Chrome tab (as described in [Firebase Setup tutorial](/overview/firebase_setup.md)), you should see your database is empty (unless you've already run some experiments before doing this tutorial).

Open up another tab and load your experiment page.  As you run through the experiment, you can refresh your Firestore console page to see there are new firebase documents being added to the `trial-metadata` collection.

Great work!  To recap, getting the PsyanimJsPsychPlugin writing data out to `Google Firebase Firestore` during an experiment was as easy as:

1) Adding a valid `firebase.config.json` to your project
2) Uncommenting a few lines of auto-generated code in `index.js` to create a firebase client and set it as a `document writer` for the `PsyanimJsPsychPlugin`
3) Copy the `projectId` value from your `firebase.config.json` into your `.firebaserc`'s "projects.default" field value

The 3rd step above is actually so that `Firebase CLI` calls (which some of the scripts in `package.json` make) will know what Firebase project to use.

## 4. PsyanimJsPsychPlugin Experiment Database Schema

Let's walk through some of the high-level structure of the Psyanim-2 database schema.

Every jsPsych trial that uses the `PsyanimJsPsychPlugin` will write out a single document to the `trial-metadata` collection which contains all of the necessary information for reproducing that particular experiment's trial.

So, the `trial-metadata` collection's documents are the most critical documents for understanding how all other data in our database is related.

If you click on one of the documents in the Firebase console and collapse all of it's fields, you'll see two top-level fields: `data` and `time`.

The top-level `data` field contains all of the data for that document and the `time` is just the timestamp when that document was last modified.

If you expand the `data` field for a given document in the `trial-metadata` collection, you'll see that there are many sub-fields.

For a detailed breakdown of the contents of the `data` field, checkout out that section in our [Database Schema](/overview/database_schema.md#_2-trial-metadata-collection) docs.

Aside from the `trial-metadata` collection, Psyanim-2 has several other types of document collections it uses such as `animation-clips`, `session-logs`, `state-logs`, etc.

We will encounter some of these in the next tutorial.

## 5. Deploying Experiments

Now that we've created and tested our experiment, it's ready for deployment on the internet where test subjects can use it to participate in the experiment.

`Psyanim-2` experiments using `jsPsych` are bundled into a single static webpage that can be deployed anywhere that will host a static site.

It can be hosted in Firebase, or Github pages, or even a custom server.

When we run `npm run build` or `npm run watch`, the experiment app is bundled by [webpack](https://webpack.js.org/) for us into the `./dist` directory in our project folder.

Inside this `./dist` folder, you'll find files generated by `webpack` from our experiment project codebase.  

There will be one `index.html` and some javascript `*.bundle.js` files.  The files are all you need for your deployment.  You can just copy them directly into your hosting service.