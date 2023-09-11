# <ins>Getting started quickly with Psyanim 2.0</ins>

Note: Full tutorial project + source code can be found [here](https://github.com/thefinnlab/psyanim-overview-tutorials/tree/hello-psyanim2).

## 1. Core concepts

Psyanim 2.0 is a framework for creating browser-based psychology / cognitive science research experiments involving 2D procedural animation.

The framework is built on top of [Phaser 3](https://phaser.io/) and has a flexible [component architecture](https://gameprogrammingpatterns.com/component.html).

In Psyanim 2.0, everything that is seen in the world lives in a `PsyanimScene`.

Any object that exists in a scene, regardless of its visual representation, is called an `PsyanimEntity`.

A `PsyanimEntity` is simply a container for `PsyanimComponents`.  

Entities may or may not have a visual representation in the scene.

Moreover, entities alone do not have any logic or behaviors.  While entities have no user-defined state, they do have a position, orientation and velocity in the world, and can have forces / accelerations applied to them.

All user-defined state and behaviors are encapsulated in `PsyanimComponents`, which can be attached to a `PsyanimEntity`.

When building experiments using [jsPsych](https://www.jspsych.org/), we can run `trials` using the `PsyanimJsPsychPlugin` where each trial runs an instance of a `PsyanimScene`.

## 2. Creating new npm project & installing Psyanim 2 and Psyanim CLI

***Pre-requisites: Requires NodeJS v18.16.0 or higher.***

***You'll need to make sure you have read access to our psyanim-2 and psyanim-cli private git repos***

Create a directory named 'hello-psyanim2' and navigate to it in your terminal.

Create a new npm project with:

```bash
npm init -y
```

Install psyanim-2 package straight from the git repo via npm with:

```bash
npm install git+https://github.com/thefinnlab/psyanim-2.git
```

Install psyanim-cli package straight from git repo via npm with:

```bash
npm install git+https://github.com/thefinnlab/psyanim-cli.git
```

## 3. Creating our first psyanim-2 experiment using the Psyanim CLI

Let's create a simple experiment to get familiar with using Psyanim 2.0.

At any time, you can run `npx psyanim-cli --help` to see the docs for Psyanim CLI.

**Optional:** Install http-server (unless you have your own static file server tool) and refer to the [docs](https://www.npmjs.com/package/http-server) to host your builds locally

```bash
npm install --global http-server
```

Note: If you're on linux / macOS, you may need to prepend 'sudo' to the start of global npm install commands and have appropriate privileges to install packages globally, such as 'http-server' above.

In the 'hello-psyanim2' directory we created in the previous step, create a new experiment using the Psyanim CLI:

```bash
npx psyanim-cli --init
```

You should now see the following files created under ./src:

- `index.js`: entry-point into the application
- `index.html`: HTML web page that will contain the Psyanim canvas
- `EmptyScene.js`: a blank Psyanim scene definition

You should also see a `.gitignore` and a `webpack.config.js` automatically generated for your experiment, and your package.json updated with some helpful commands for building with webpack, deploying to firebase, etc.

Let's go ahead and initialize a git repository and commit what we have so far!

```bash
git init
git add .
git commit -m "created my first psyanim experiment!"
```

At this stage, we have an experiment with just one empty scene.  Let's add another scene.

# 4. Creating our first scene using Psyanim CLI

Back in the terminal, run the following command to create a scene named 'MyFirstScene':

```bash
npx psyanim-cli -s MyFirstScene
```

You should see `MyFirstScene.js` show up under `./src/`.

Let's add this new scene to our `index.js` file.

Open `index.js` and add the following import to the top of the file:

```js
import MyFirstScene from './MyFirstScene';
```

Let's register `MyFirstScene` with `PsyanimApp` by adding the following line right before `Psyanim.Instance.run()`:

```js
PsyanimApp.Instance.config.registerScene(MyFirstScene);
```

Next, let's declare a trial `myfirstSceneTrial` after the `emptySceneTrial` is declared by adding the following code after it:

```js
let myFirstSceneTrial = {
    type: PsyanimJsPsychPlugin,
    sceneKey: MyFirstScene.key,
};
```

Now let's add this new trial we defined to the line with `jspsych.run()`, by replacing it with the following line:

```js
jsPsych.run([welcome, emptySceneTrial, myFirstSceneTrial, goodbye]);
```

**After these modifications, your `index.js` should look like [this](https://github.com/thefinnlab/psyanim-overview-tutorials/blob/hello-psyanim2/src/index.js).**

Run `npm run build` in your terminal and then look in the ./dist directory to see your index.html.  Load this in your browser using whatever static file server you prefer.

- If you hit `F12` to open the chrome debug tools, you'll be able to see the console output from the app.
- To load the `EmptyScene` and `MyFirstScene` trials we added to jsPsych, just hit `Enter` on your keyboard.
- You should see the console say what scenes are loaded as you move through the trials.

**Congratulations, you've created your first experiment with a user-defined scene!**

# 5. Creating our first entity and component using the Psyanim CLI

Back in our terminal, let's create our first component `MyFirstMovementComponent` by running the following command:

```bash
npx psyanim-cli -c MyFirstMovementComponent
```

You should see `MyFirstMovementComponent.js` under `/src/`.

Open up `MyFirstScene.js` and add the following code so the imports at the top and your `scene definition` looks like the following:

```js
import { PsyanimConstants } from 'psyanim2';

import MyFirstMovementComponent from './MyFirstMovementComponent';

export default {
    key: 'MyFirstScene',
    wrapScreenBoundary: false,
    entities: [
        {
            name: 'agent1',
            initialPosition: { x: 400, y: 300 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE, 
                radius: 10, 
                color: 0xff0000
            },
            components: [
                { type: MyFirstMovementComponent }
            ]
        }
    ]
}
```

To add an `entity` in the `scene definition` above, we add a single `entity definition` to our `entities` array called `agent1`.

The `agent1` entity has a red circle representation in the scene and is centered at pixel coordinates `(400, 300)` in the scene.

Notice we add our `MyFirstMovementComponent` to our `agent1` entity by adding a single object to its `components` array property.  The `type` field of the `component definition` object specifies the class type of the `component` we want to add.

Open MyFirstMovementComponent.js and add the following code to have the agent move back and forth horizontally:

```js
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
```

The `update` method above is called every frame (60 times per second).

Inside the `update` method, we've added code to compute the velocity of the entity, then compute it's displacement from it's velocity and the `dt` (or 'delta time') parameter supplied to `update`, and then add that displacement to the entity's current position.

Note that using `this.entity` within a component will return a reference to the entity which the component is attached to.

Rebuild your ./dist bundle and reload the page in your browser and you should be able to see your agent moving back and forth on-screen by using the 'enter' key to load the experiment scenes!

<p align="center">
  <img src="./imgs/getting_started_final_result.gif" />
</p>

- Remember to watch the console to see what scene is loaded when you are in a `PsyanimJsPsychPlugin` trial.

**Congratulations, you've created your first movement component and attached it to an entity in the scene to control its movement!**

In this tutorial, we learned about how to create scenes with entities in Psyanim 2.0 and add components to those entities to give them state and behavior.

While writing custom components can give you full control over an entity's state & behavior, it may not be necessary, depending on the needs of your experiment.

Psyanim 2.0 comes with several components for player control and AI steering behaviors, so head over to the [next tutorial](/overview/scenes_and_entities.md) to see how to leverage them for your experiments.