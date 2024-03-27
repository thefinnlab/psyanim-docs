# <ins>Passive Viewing Example</ins>

## 1. Overview and Project Setup

Sometimes, an experiment may not require any control input from the user and is designed to only be observed passively.

In these instances, researchers have even more control over the quality and types of agent-based animations that are presented to the user, since many stochastic simulations can be generated offline and hand-picked for quality control before experiments are deployed.

The workflow for experiments involving only passive observation by test subjects involves the following steps, at a high-level:

1. Create an experiment to run simulations and save agent trajectories to Firebase.
2. Use Psyanim Experiment Viewer to view and select desired trials from Firebase.
3. Create another experiment to deploy with selected trials to playback for test subjects.

In `step 1` above, the experiment you'll create is specifically designed for the purpose of generating many trials and trajectories with desired variations.

In `step 2`, you'll QC the data generated from your simulations and select only trials with satisfactory trajectories, saving them to a `Trial Collection File`.

In `step 3`, the experiment you'll create is the one that you'll deploy online for test subjects to participate in, playing back trials from the `Trial Collection File` you created in `step 2`.

---

In this project, we'll create a `chase-subtlety` experiment where a `predator` agent will wander around, periodically chasing after a `prey` agent in the scene.

To setup our experiment project for `step one`, we need to create a new psyanim-2 project and create a `Predator-Prey v2` scene to generate simulation data with.

A template project for the `predator-prey v2` algorithm is already provided by `Psyanim-CLI` for us, so all we have to do is create a directory called `passiveViewingSimulator`, navigate to that directory and run the following command:

```bash
psyanim init -h
```

You'll see a list of `Sub-Commands` for the psyanim-cli `init` command, and the one we're interested in is `predprey`, which generates a template project for predator-prey v2 experiment.

To generate our `predator-prey v2` template project, run the following command:

```bash
psyanim init:predprey
```

In a different terminal, start a watch service from the project root with `npm run watch`.

In another terminal, start a local development server from the project root with `npm run serve`.

Open a chrome browser and navigate to `localhost:3000` and your jsPsych experiment should have 1 psyanim-2 trial, with a predator and prey agent in it.

// TODO: add video

## 2. Simulating Agent Trajectories

Now that we've created an experiment project to simulate agent trajectories, we just need to set this project up to save those trajectories to firebase.

We'll also add some trial variations to make this a more realistic experiment, where we may have lots of variations we generate to show different users.

---

First, let's copy our `firebase.config.json` into the project root directory.

Next, in the experiment project we created in the previous section, open up `./src/index.js` and make some modifications.

Update the `psyanim-2` import statement at the top of the file so it includes the `PsyanimFirebaseBrowserClient` and `PsyanimPredatorFSM`, as follows:

```js
import { 
    PsyanimApp, 
    PsyanimJsPsychPlugin,
    PsyanimJsPsychTrial,

    PsyanimFirebaseBrowserClient,
    PsyanimPredatorFSM

} from 'psyanim2';
```

We also need to import our Firebase config object by adding the following line to the end of our imports at the top of `index.js`:

```js
import FirebaseConfig from '../firebase.config.json';
```

Next, we need to create a Firebase client and set it as the `document writer` for the `PsyanimJsPsychPlugin` by adding the following lines to the end of the section in code with the comment `Setup PsyanimJsPsychPlugin`:

```js
/**
 *  Setup PsyanimJsPsychPlugin
 */
PsyanimJsPsychPlugin.setUserID(userID);
PsyanimJsPsychPlugin.setExperimentName(experimentName);

let firebaseClient = new PsyanimFirebaseBrowserClient(FirebaseConfig);
PsyanimJsPsychPlugin.setDocumentWriter(firebaseClient);
```

We also need to tell the `PsyanimJsPsychPlugin` which agents to record in each trial, or it won't record any trajectory data for that trial.

To do so, add a call to `addAgentNamesToRecord()` on our predator-prey trial object before we push it into the `jsPsych timeline`, as follows:

```js
// predator-prey scene trial
let predatorPreySceneTrial = new PsyanimJsPsychTrial(PredatorPreyV2, PredatorPreyV2.key);

// tell PsyanimJsPsychPlugin to record the 'predator' and 'prey' agents
predatorPreySceneTrial.addAgentNamesToRecord(['predator', 'prey']);

timeline.push(predatorPreySceneTrial.jsPsychTrialDefinition);
```

Finally, let's create a few trial variations by wrapping the code above in a loop and updating the `subtlety` parameter of the `PsyanimPredatorFSM` component as follows:

```js
let subtletyValues = [5, 45, 135];

for (let i = 0; i < subtletyValues.length; ++i)
{
    // predator-prey scene trial
    let predatorPreySceneTrial = new PsyanimJsPsychTrial(PredatorPreyV2, 'subtlety_' + (i + 1));

    predatorPreySceneTrial.setComponentParameter(
        'predator', PsyanimPredatorFSM, 'subtlety', subtletyValues[i]);

    predatorPreySceneTrial.addAgentNamesToRecord(['predator', 'prey']);

    timeline.push(predatorPreySceneTrial.jsPsychTrialDefinition);
}
```

Notice that I completely changed the `scene key` for each trial.  This is because each trial must have a unique scene key and there's no need for the scene keys to be related to the key of the original `scene definition` template we used.

Reload the experiment in your browser (at `localhost:3000`) and run through the experiment at least once, so you'll have some trial data saved in Firebase.

To verify your data got saved fine in Firebase, navigate to your [Firebase Console](https://console.firebase.google.com/) in your web browser, and then navigate to the `Cloud Firestore` section.

In the database table in the GUI, you should see two collections:

- `trial-metadata`
- `animation-clips`

Click on those collections to verify they have the correct number of documents.

If this was a fresh Firebase project that you just setup, your database would have been empty, and thus a quick sanity check is there should be 2 `animation clip` documents for every `trial-metadata` document in your database.

Before you proceed, make sure you ran through your experiment in the browser at least 1 time and that you have at least 3 valid trial metadata documents and 6 animation clip documents.

If you don't see these, then run through your experiment & code one more time to make sure you have trajectories to use in the next section.

Great work - you've setup a predator-prey v2 experiment project that you can use to generate trajectories to be used for passive viewing in other online experiments.

## 3. Quality-Control with Experiment Viewer

In the previous section, we created an experiment project to generate agent trajectories that we saved to Firebase so that we can QC these simulated trajectories before deploying them in an online experiment.

The next step in this process is to select the simulated trajectories which satisfy the needs of our current experiment.

Not all simulated trajectories will meet the needs of each experiment, so it's necessary to inspect them visually and manually filter out the ones we aren't interested in.

To do this, a tool called `Psyanim Experiment Viewer` has been developed with two goals in mind:

1. Provide researchers with an easy way to playback trajectories stored in Firebase
2. Provide researchers with a means to select trajectories to add to a `Trial Collection File`, in context with visualization

// TODO: need to add section on how to use psyanim-cli to do firebase queries, using sample queries to demonstrate!