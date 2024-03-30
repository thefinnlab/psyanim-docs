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

In the previous section, we created an experiment project to generate agent trajectories (`step 1`) that we saved to Firebase so that we can QC these simulated trajectories (`step 2`) before deploying them in an online experiment (`step 3`).

The next step in this process is to select the simulated trajectories which satisfy the needs of our current experiment.

Not all simulated trajectories will meet the needs of each experiment, so it's necessary to inspect them visually and manually filter out the ones we aren't interested in.

To do this, a tool called `Psyanim Experiment Viewer` has been developed with two goals in mind:

1. Provide researchers with an easy way to playback trajectories stored in Firebase for visual analysis
2. Provide researchers with a means to select trajectories to add to a `Trial Collection File`, in context with visualization

--- 

To get started, let's clone the [Psyanim Experiment Viewer](https://github.com/thefinnlab/psyanim-experiment-viewer) repository using the following command from a parent directory in your terminal:

```bash
git clone https://github.com/thefinnlab/psyanim-experiment-viewer.git
```

Navigate into the directory of the git repo you just cloned and copy your `service-account.json` file for your `Firebase Firestore` project into the root of your local experiment viewer repository.

This `service-account.json` file allows Psyanim Experiment Viewer to access the data in your Firestore database.

Psyanim Experiment Viewer has a client-server architecture, so part of it will run in `nodejs` and the other part of the software will run in your `Chrome browser`.

The part running in `nodejs` is referred to as the `server` and the part running in the `Chrome browser` is referred to as the `client`.

To run `psyanim experiment viewer`, we must first build the client with the following command in terminal:

```bash
npm run build
```

Then, we just need to start the server with the following command:

```bash
npm run serve
```

Now, you should be able to navigate to `localhost:7000` in your Chrome browser to run the experiment viewer app.

You should be able to view the animations for trials in your database using `j` and `k` keys to cycle through them and `spacebar` to restart the current animation.

---

Note that the experiment viewer app server loads trial + animation clip data from firestore when you start the server (using the `npm run serve`) command.

By default, experiment viewer's server will load all trial data from Firestore when you start it.

If the data in your database changes, or you want to load a different subset of the data for viewing, you'll need to `restart` your experiment viewer server.

To restart the app server, kill the server process in your terminal (with `ctrl+c`) and then start it with `npm run serve` again.

If you want to load just a subset of the data from your Firestore database for viewing, you can create a `custom dataprovider` to filter out what data experiment viewer should load.

We can also create custom queries to run against our Firebase database using `Psyanim-CLI` to quickly add/edit or retrieve data from firebase via a simple nodejs script.

In the next sections, we'll learn how to create and perform custom queries with `Psyanim-CLI` and how to create `custom dataproviders` to filter data loaded by experiment viewer.

## 4. Firebase Query Scripts

`Psyanim-CLI` has the ability to run custom `query scripts` against your Firebase Firestore instances, allowing users to read, edit, and write data to/from Firebase cloud instances.

In this section, we'll write some existing sample data on our local disk out to our Firebase instance.

The sample data is provided out-of-the-box for testing with the experiment viewer app.

The data contains `trial-metadata` & `animation-clip` collections that we can use in our experiments.

Run the following command in your terminal to create a custom query script that we can add our own logic to:

```bash
psyanim asset:query -o ./queries restoreCloudDB
```

The query script that's auto-generated comes with some example code of how to process args/options in your script, as well as how to use the `db` instance from the [Firebase Admin SDK](https://firebase.google.com/docs/admin/setup).

---

Let's go ahead and delete the body of the `psyanimCliQuery()` function, so it looks like this:

```js
export function psyanimCliQuery(db, args, options) {

};
```

This is the basic form of a firebase query in psyanim-cli.  The query lives in a `query script` nodejs module that should export a single function named `psyanimCliQuery`.

That function should accept 3 arguments:

- `db`: A database instance from the `Firebase Admin SDK`
- `args`: A list of command-line arguments for the script
- `options`: A dictionary of options

Add the following imports at the top of the query script file, before we export the `psyanimCliQuery()` function:

```js
import fs from 'fs';
import path from 'path';
```

Add the following code to the body of the `psyanimCliQuery()` function:

```js
export function psyanimCliQuery(db, args, options) {

    if (args.length == 0)
    {
        console.error("ERROR: no database file path provided!");
        return;
    }

    let rootDir = path.resolve(args[0]);

    let loadCollection = (collectionName) => {

        let docObjects = [];
    
        let collectionDir = path.join(rootDir, collectionName);
    
        if (!fs.existsSync(collectionDir))
        {
            console.warn("Directory doesn't exist: ", collectionDir);
            return null;
        }
    
        fs.readdirSync(collectionDir)
            .forEach((fileName) => {
    
                let docFilePath = path.resolve(path.join(collectionDir, fileName));
    
                let docId = path.parse(fileName).name;
    
                let docData = JSON.parse(fs.readFileSync(docFilePath));
    
                docObjects.push({
                    id: docId,
                    docData: docData
                });
            });
    
        return docObjects;
    }
    
    let writeCollectionToFirestore = (db, collectionName, docObjects) => {
    
        docObjects.forEach((docObject) => {
            db.collection(collectionName)
                .doc(docObject.id)
                .set(docObject.docData);
        });
    };
    
    let pushLocalCollectionToFirestore = (db, collectionName) => {
    
        let docObjects = loadCollection(collectionName);
    
        if (!docObjects) {
    
            console.warn('Failed to push collection to firestore: ', collectionName);
            return;
        }
    
        writeCollectionToFirestore(db, collectionName, docObjects);
    };
    
    pushLocalCollectionToFirestore(db, 'trial-metadata');
    pushLocalCollectionToFirestore(db, 'animation-clips');
    pushLocalCollectionToFirestore(db, 'state-logs');
    pushLocalCollectionToFirestore(db, 'session-logs');
    pushLocalCollectionToFirestore(db, 'jspsych-experiment-data');
};
```

This query script accepts one command-line parameter, which is a path to a dataset on the local disk to restore to the cloud databse.

It loads the Firestore documents from the local disk and writes them to the cloud via the `db` instance passed to the `psyanimCliQuery()` function.

To run this script and push the sample-data to your firebase instance, run the following command from the root of your experiment-viewer project:

```bash
psyanim firebase:query ./queries/restoreCloudDB.js ./sample_firestore_datasets/2-experiments-5-users
```

The first argument is for psyanim-cli to locate the query script file to run and the second argument is passed on to our query script to process.

The second argument is the path to the dataset on disk that we'd like to upload to the cloud firestore database.

Take a look at your [Firebase Console](https://console.firebase.google.com/) `Cloud Firestore` GUI to verify you see new documents in your `trial-metadata` and `animation-clips` collections.

---

Let's restart experiment viewer's server and then navigate to `localhost:7000` in our Chrome browser to view the data.

As you browse the various trials that are loaded, you should notice the following user IDs in the data-set: `UserA`, `UserB`, `UserC`, `UserD`, `UserE`, and `Jason`.

`Jason` is the user ID from the passive-viewing experiment we created and ran.

`Users A-E` are from the sample data-set we just uploaded to Firebase using our `restoreCloudDB.js` query script.

Great work!  You created your first `custom query script` to upload data to Firebase from your local disk and verified it using `Firebase console` as well as `Psyanim Experiment Viewer`!

// TODO: add video walkthrough

## 5. Custom Data Providers

Now that we've got data from 6 different users in our Firebase cloud database, we can take this opportunity to write our first `custom dataprovider` to only load a subset of the data in our Firebase instance for viewing in the experiment viewer app.

Remember that, by default, the experiment viewer app will load all documents in our `trial-metadata` collection from Firebase, and this could potentially be an enormous amount of data.

Instead of loading it all, it's best to create a `custom dataprovider` script that filters what subset of data we want to load from the cloud DB.

Note that writing a custom dataprovider script requires a bit of familiarity or comfort with navigating the Firebase Firestore APIs, linked further down in this tutorial section.

For most types of queries, the APIs are fairly straight forward and easy-to-use.  The dataprovider scripts auto-generated by psyanim-cli also show a few examples of how to use and chain filters in your custom queries.

---

To create a custom dataprovider script, run the following command in your terminal:

```bash
psyanim asset:dataprovider myUserACEJasonDataProvider -o ./dataproviders
```

Open up the `myUserACEJasonDataProvider.js` under the `./dataproviders` directory and let's take a peek at its contents.

The reason we named this script `myUserACEJasonDataProvider.js` is that we want the dataprovider to only pull data for the following user IDs: `UserA`, `UserC`, `UserE`, and `Jason`.

A custom data provider exports a single function, `dataProvider()`, which accepts two arguments.

The first argument is a firebase SDK client reference, where `firebaseClient.db` is a Firestore DB object reference (see: https://firebase.google.com/docs/firestore/query-data/queries).

The second argument is a `dataHandler` delegate, which Psyanim Server expects your `dataProvider()` function to call, passing it an array of valid firebase document references.

Replace the contents of your `myUserACEJasonDataProvider.js` dataprovider script with the following code:

```js
export function dataProvider(firebaseClient, dataHandler) {

    let userIDsToLoad = ['UserA', 'UserC', 'UserE', 'Jason'];

    firebaseClient.db
        .collection('trial-metadata') // get documents in 'trial-metadata' collection
        .where('data.userID', 'in', userIDsToLoad) // only load docs with user ID in 'userIDsToLoad' array
        .get() // perform the query
        .then((querySnapshot) => {

            // when the query is complete, pass the docs returned in the querySnapshot to our 'dataHandler'
            dataHandler(querySnapshot.docs);
        });
};
```

Notice that, when you save the dataprovider *.js file, experiment viewer will automatically restart.  Any time you change a file in the experiment viewer project, the app should restart (for most files, with a few exceptions).

That said, even though experiment viewer restarted, it isn't pointing to the correct data provider yet.

To do so, we'll need to stop the server and restart it with a slightly modified command:

```bash
npm run serve -- ./dataproviders/myUserACEJasonDataProvider.js
```

The command above is just like the original command used to start the experiment viewer server, but with a `--` followed by a path to the custom dataprovider script that we want it to use.

This time, experiment viewer will load the `myUserACEJasonDataProvider.js` at start-up and run the query in the `dataProvider()` function we provided it.

Reload the experiment viewer app in your Chrome browser (at `localhost:7000`) and you should see that you now only have data for user IDs: `UserA`, `UserC`, `UserE`, and `Jason`.

Great work - you've written your first custom dataprovider for Psyanim Experiment Viewer!

There are many ways you can filter the data, and even do local filtering, but the fundamental workflow doesn't change:

1. Use `psyanim-cli` to generate boiler plate for data provider
2. Implement logic for data provider
3. Start Experiment Viewer server, pointing it to the location of the custom data provider.

// TODO: add video walkthrough

## 6. Selecting Trials for Playback

In the previous sections, we created an experiment project to simulate `predator-prey v2` interactions and save the results of those experiment runs off to Firebase.

We also added some extra test data to our Firebase data from the experiment viewer's sample datasets.

In this section, we'll create a new experiment project and treat it as if we are actually going to deploy it online for test subjects to participate in.

In this new experiment, we'll playback previously recorded trials selected from those that already exist in our Firebase cloud instance.

To select which trials we want to include for playback in this new experiment, we'll use `Psyanim Experiment Viewer` to view previously recorded trials and save them to a `Trial Collection File`.

A `Trial Collection File` contains references to a collection of trials that exist in Firebase.  You can then playback all the trials in a `Trial Collection File` using a `PsyanimJsPsychExperimentPlayer` component.

Before we proceed to setting up the new experiment project, let's use the experiment viewer app to select which trials we want in our upcoming experiment.

Navigate to your local experiment viewer app repo in a terminal and let's create a new custom data provider to filter our selection of trials down to just those from `UserA` and `Jason`.

To do so, run the following command in your terminal:

```bash
psyanim asset:dataprovider myUserAJasonDataProvider -o ./dataproviders
```

Next, replace the contents of `myUserAJasonDataProvider.js` with the following code:

```js
export function dataProvider(firebaseClient, dataHandler) {

    let userIDsToLoad = ['UserA', 'Jason'];

    firebaseClient.db
        .collection('trial-metadata') // get documents in 'trial-metadata' collection
        .where('data.userID', 'in', userIDsToLoad) // only load docs with user ID in 'userIDsToLoad' array
        .get() // perform the query
        .then((querySnapshot) => {

            // when the query is complete, pass the docs returned in the querySnapshot to our 'dataHandler'
            dataHandler(querySnapshot.docs);
        });
};
```

Now, we can start the experiment viewer server with the dataprovider we just created, using the following command:

```bash
npm run serve -- ./dataproviders/myUserAJasonDataProvider.js
```

// TODO: user needs to open app in browser and add trials to collection file