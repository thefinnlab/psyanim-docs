# Getting started with Psyanim 2.0

## 1. Core concepts

Psyanim 2.0 is a framework for creating browser-based psychology / cognitive science research experiments involving 2D procedural animation.

The framework is built on top of [Phaser 3](https://phaser.io/) and has a flexible [component architecture](https://gameprogrammingpatterns.com/component.html).

In Psyanim 2.0, everything that is seen in the world lives in a `PsyanimScene`.

Any object that exists in a scene, regardless of its visual representation, is called an `PsaynimEntity`.

A `PsyanimEntity` is simply a container for `PsyanimComponents`.  

Entities may or may not have a visual representation in the scene.

Moreover, entities alone do not have any logic or behaviors.  While entities have no user-defined state, they do have a position, orientation and velocity in the world, and can have forces / accelerations applied to them.

All user-defined state and behaviors are encapsulated in `PsyanimComponents`, which can be attached to a `PsyanimEntity`.

When building experiments using [jsPsych](https://www.jspsych.org/), we can run `trials` using the `PsyanimJsPsychPlugin` where each trial runs an instance of a `PsyanimScene`.

## 2. Creating new npm project & installing Psyanim 2 and Psyanim CLI

***Pre-requisites: Requires NodeJS v18.16.0 or higher.***

***You'll need to make sure you have read access to our psyanim-2 and psyanim-cli private git repos***

**a. Create a directory named 'hello-psyanim2' and navigate to it in your terminal.**

**b. Create a new npm project with:**

    npm init -y

**c. Install psyanim-2 package straight from the git repo via npm with:**

    npm install git+https://github.com/thefinnlab/psyanim-2.git

**d. Install psyanim-cli package straight from git repo via npm with:**

    npm install git+https://github.com/thefinnlab/psyanim-cli.git

## 3. Creating our first psyanim-2 experiment using the Psyanim CLI

Let's create a simple experiment to get familiar with using Psyanim 2.0.

At any time, you can run `npx psyanim-cli --help` to see the docs for Psyanim CLI.

**Optional:** Install http-server (unless you have your own static file server tool) and refer to the [docs](https://www.npmjs.com/package/http-server) to host your builds locally

    npm install --global http-server

**a. In the 'hello-psyanim2' directory we created in the previous step, create a new experiment using the Psyanim CLI:**

    npx psyanim-cli --init

You should now see the following files created under ./src:

- `EmptyScene.js`
- `index.html`
- `index.js`

You should also see a `.gitignore` and a `webpack.config.js` automatically generated for your experiment, and your package.json updated with some helpful commands for building with webpack, deploying to firebase, etc.

**b. Let's go ahead and initialize a git repository and commit what we have so far!**

    git init
    git add .
    git commit -m "created my first psyanim experiment!"

At this stage, we have an experiment with just one empty scene.  Let's add another scene.

# 4. Creating our first scene using Psyanim CLI

**a. Back in the terminal, run the following command to create a scene named 'MyFirstScene':**

    npx psyanim-cli -s MyFirstScene

You should see `MyFirstScene.js` show up under `./src/`.

Let's add this new scene to our `index.js` file.

**b. Open `index.js` and add the following import to the top of the file:**

    import MyFirstScene from './MyFirstScene';

**c. Let's register `MyFirstScene` with `PsyanimApp` by adding the following line right before `Psyanim.Instance.run()`:**

    PsyanimApp.Instance.config.registerScene(MyFirstScene);

**d. Next, let's declare a trial `myfirstSceneTrial` after the `emptySceneTrial` is declared by adding the following code after it:**

    let myFirstSceneTrial = {
        type: PsyanimJsPsychPlugin,
        sceneKey: MyFirstScene.KEY,
        experimentName: experimentName,
        userID: userID,
        sceneParameters: { }
    };

**e. Now let's add this new trial we defined to the line with `jspsych.run()`, by replacing it with the following line:**

    jsPsych.run([welcome, emptySceneTrial, myFirstSceneTrial, goodbye]);

**After these modifications, your `index.js` should look like the following:**

    import { initJsPsych } from 'jspsych';

    import htmlKeyboardResponse from '@jspsych/plugin-html-keyboard-response';

    import { PsyanimApp, PsyanimJsPsychPlugin } from 'psyanim2';

    import EmptyScene from './EmptyScene';
    import MyFirstScene from './MyFirstScene';

    /**
    *  Setup Psyanim and PsyanimJsPsychPlugin
    */
    PsyanimApp.Instance.config.registerScene(EmptyScene);
    PsyanimApp.Instance.config.registerScene(MyFirstScene);

    PsyanimApp.Instance.run();

    PsyanimApp.Instance.setCanvasVisible(false);

    /**
    *  Setup jsPsych experiment
    */

    const userID = 'Jason';
    const experimentName = 'defaultExperimentName';

    const jsPsych = initJsPsych();

    let welcome = {
        type: htmlKeyboardResponse,
        stimulus: 'Welcome to the experiment.  Press any key to begin.'
    };

    let emptySceneTrial = {
        type: PsyanimJsPsychPlugin,
        sceneKey: EmptyScene.KEY,
        experimentName: experimentName,
        userID: userID,
        sceneParameters: { },
    };

    let myFirstSceneTrial = {
        type: PsyanimJsPsychPlugin,
        sceneKey: MyFirstScene.KEY,
        experimentName: experimentName,
        userID: userID,
        sceneParameters: { }
    };

    let goodbye = {
        type: htmlKeyboardResponse,
        stimulus: 'Congrats - you have completed your first experiment!  Press any key to end this trial.'
    };

    jsPsych.run([welcome, emptySceneTrial, myFirstSceneTrial, goodbye]);

**f. Run `npm run build` in your terminal and then look in the ./dist directory to see your index.html.  Load this in your browser using whatever static file server you prefer.**

- If you hit `F12` to open the chrome debug tools, you'll be able to see the console output from the app.
- To load the `EmptyScene` and `MyFirstScene` trials we added to jsPsych, just hit `Enter` on your keyboard.
- You should see the console say what scenes are loaded as you move through the trials.

**Congratulations, you've created your first experiment with a user-defined scene!**

# 5. Creating our first entity and component using the Psyanim CLI

**a. Back in our terminal, let's create our first component `MyFirstMovementComponent` by running the following command:**

    npx psyanim-cli -c MyFirstMovementComponent

You should see `MyFirstMovementComponent.js` under `/src/`.

**b. Open up `MyFirstScene.js` and add the following code so the imports at the top and your `create` method looks like the following:**

    import Phaser from 'phaser';

    import 
    { 
        PsyanimScene,
        PsyanimConstants,

    } from 'psyanim2';

    import MyFirstMovementComponent from './MyFirstMovementComponent';

    export default class MyFirstScene extends PsyanimScene {

        static KEY = 'MyFirstScene';

        constructor() {

            super(MyFirstScene.KEY);
        }

        init() {
            
            super.init();
        }

        preload() {
            
            super.preload();
        }

        create() {

            super.create();

            let agent1 = this.addEntity('agent1', 400, 300, {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE, 
                radius: 10, 
                color: 0xff0000
            });

            agent1.addComponent(MyFirstMovementComponent);
        }

        update(t, dt) {

            super.update(t, dt);
        }
    }

In the `create` method above, we add an entity called `agent1`.

The `agent1` entity has a red circle representation in the scene and is centered at pixel coordinates `(400, 300)` in the scene.

Notice we add our `MyFirstMovementComponent` to our `agent1` entity with a simple call to `addComponent`.

**c. Open MyFirstMovementComponent.js and add the following code to have the agent move back and forth horizontally:**

    import Phaser from 'phaser';

    import { PsyanimComponent } from 'psyanim2';

    export default class MyFirstMovementComponent extends PsyanimComponent {

        speed;

        constructor(entity) {

            super(entity);

            this.speed = 0.5;
        }

        update(t, dt) {

            super.update(t, dt);

            let oldPosition = this.entity.position;

            // compute speed
            if (oldPosition.x > 700 && this.speed > 0)
            {
                this.speed *= -1.0;
            }
            else if (oldPosition.x < 100 && this.speed < 0)
            {
                this.speed *= -1.0;
            }

            // compute displacement over 'dt'
            let displacement = new Phaser.Math.Vector2(this.speed * dt, 0);

            // compute new position and set this entity's position to it
            oldPosition.add(displacement);

            this.entity.position = oldPosition;
        }
    }

The `update` method above is called every frame (60 times per second).

Inside the `update` method, we've added code to compute the speed of the entity, then compute it's displacement from it's speed and the `dt` (or 'delta time') parameter supplied to `update`, and then add that displacement to the entity's current position.

Note that using `this.entity` within a component will return a reference to the entity which the component is attached to.

**d. Rebuild your ./dist bundle and reload the page in your browser and you should be able to see your agent moving back and forth on-screen by using the 'enter' key to load the experiment scenes!**

- Remember to watch the console to see what scene is loaded when you are in a `PsyanimJsPsychPlugin` trial.

**Congratulations, you've created your first movement component and attached it to an entity in the scene to control its movement!**