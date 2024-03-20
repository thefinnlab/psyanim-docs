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
let interactivePredatorMouseV2Trial = new PsyanimJsPsychTrial(
    InteractivePredatorMouseV2, InteractivePredatorMouseV2.key);

timeline.push(interactivePredatorMouseV2Trial.jsPsychTrialDefinition);
```

// TODO: notes on watch/serve and running in browser + what to expect