# <ins>Interactive Predator Chasing</ins>

## 1. Overview and Project Setup

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
psyanim template:predatormousev2 -o .\src\scenes\
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

Reload the experiment in your browser and you should see the green predator agent wandering about, occasionally turning red and charging after the mouse cursor before returning to a wander.

// TODO: add video!

<!-- <p align="center" style="font-size: 12px;">
    <video width="640" height="360" controls>
    <source src="./videos/test.mp4" type="video/mp4">
    </video>
</p> -->

Great work -  you've setup an interactive predator-chase experiment!

## 2. Creating Trial Variations

So far, we've seen how we can create `jsPsych Trials` from `Psyanim Scene Definitions`.

In this section, we'll create multiple `jsPsych Trials` that are variations of the same `Psyanim Scene Definition.`

This is useful for experiments where we would like to see how a subject's perception of agent interactions changes with respect to a particular variable.

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

Notice that we modified the second parameter to the `PsyanimJsPsychTrial` constructor os it passes the original scene's key, but with a '2' appended to it.

This is because the first argument to the `PsyanimJsPsychTrial` constructor is the scene definition object for the trial, while the second argument is a `unique key` associated with that scene and it's parameter-set.

So long as that `scene key` is different for every trial object we push into our timeline, we can reuse the same `scene definition` for as many trial objects as we'd like, allowing us to create multiple variations of the same scene.

To modify the parameters of the scene definition for the second trial, the trial object exposes a set of API methods that allow us to modify them and specify whether or not we want them to be recorded (we will discuss parameter recording in another tutorial).

For this trial, we use the `setEntityShapeParameter()` method, which allows us to modify any of the `shape parameters` of any `entity` in our scene definition.

For this trial, we'll change the `shapeType` of our `predator` entity to be a circle, and specify it's `radius` to be 12px.

---

Let's create one more `predator-chase` trial by adding the following code after the second trial is pushed into the jsPsych timeline:

```js
// third predator trial
let interactivePredatorMouseV2Trial3 = new PsyanimJsPsychTrial(InteractivePredatorMouseV2, InteractivePredatorMouseV2.key + '_3');

interactivePredatorMouseV2Trial3.setComponentParameter('predator', PsyanimPredatorFSM, 'maxWanderSpeed', 5.0);
interactivePredatorMouseV2Trial3.setComponentParameter('predator', PsyanimPredatorFSM, 'maxChargeSpeed', 5.0);

timeline.push(interactivePredatorMouseV2Trial3.jsPsychTrialDefinition);
```

Here, we modify two component parameters on the `predator` entity's `PsyanimPredatorFSM` components: `maxWanderSpeed` and `maxChargeSpeed`.