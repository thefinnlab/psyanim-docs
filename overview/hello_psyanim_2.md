# <ins>Getting Started with Psyanim 2.0</ins>

Note: Full tutorial project + source code can be found [here](https://github.com/thefinnlab/hello-psyanim2).

## 1. Core concepts

Psyanim 2.0 is a framework for creating browser-based psychology / cognitive science research experiments involving 2D procedural animation.

The framework is built on top of [Phaser 3](https://phaser.io/) and has a flexible [component architecture](https://gameprogrammingpatterns.com/component.html).

In Psyanim 2.0, everything that is seen in the world lives in a `PsyanimScene`, which is an abstraction for the 2D world we are simulating.

The `PsyanimScene` acts as a container for `PsyanimEntity` objects.

Any object that exists in a scene, regardless of its visual representation, is called a `PsyanimEntity`.

A `PsyanimEntity` acts a container for `PsyanimComponents`.  

More technically, it's an abstraction for anything that exists in a Psyanim Scene with a particular location, rotation, and (optionally) a visual representation which can have physics applied to it.

Entities may or may not have a visual representation in the scene.

Moreover, entities alone do not have any logic or behaviors.  While entities have no user-defined state, they do have a position, orientation and velocity in the world, and can have forces / accelerations applied to them.

Any object in your simulation that needs a position or orientation, a visual representation in the scene, or needs to have physics applied to it, should be added to the scene as an entity.

All user-defined state and behaviors are encapsulated in `PsyanimComponents`, which are reusable scripts that can be attached to a `PsyanimEntity` and offer hooks into the real-time update loop of each scene.

When building experiments using [jsPsych](https://www.jspsych.org/), we can run `trials` using the `PsyanimJsPsychPlugin` where each trial runs an instance of a `PsyanimScene`.

## 2. Creating a new Psyanim 2 Project

***Pre-requisites: Requires NodeJS v18.16.0 or higher.***

***You'll need to make sure you have read access to our psyanim-2 and psyanim-cli private git repos***

Create a directory named 'hello-psyanim2' and navigate to it in your terminal.

Create a new npm project with:

```bash
npm init -y
```

Install psyanim-2 package from the git repo via npm with:

```bash
npm install git+https://github.com/thefinnlab/psyanim-2.git
```

Install psyanim-cli package globally from git repo via npm with:

```bash
npm install -g git+https://github.com/thefinnlab/psyanim-cli.git
```

Create a new Psyanim-2 project with psyanim-cli by running the following command from the root of your project directory:

```bash
psyanim --init
```

Let's go ahead and initialize a git repository and commit what we have so far!

```bash
git init
git add .
git commit -m "created my first psyanim experiment!"
```

At this stage, we have an experiment with just one empty scene.  Let's add another scene.

## 3. Creating our first scene using psyanim-cli

Let's get familiar with `Psyanim Scenes`, `Psyanim Entities` and `Psyanim Components` by building a simple scene with a red circle entity that moves back and forth as if patrolling between two points.

Back in the terminal, run the following command to create a scene named 'MyFirstScene':

```bash
psyanim --scene MyFirstScene
```

You should see `MyFirstScene.js` show up under `./src/`.

Let's add this new scene to our `index.js` file.

Open `index.js` and add the following import to the top of the file:

```js
import MyFirstScene from './MyFirstScene.js';
```

We can also delete all commented-out code and unused imports - they are only there as an example of how to setup features we won't discuss right now.

Now, let's delete `EmptyScene.js`, remove the `emptySceneTrial` from `index.js` and create a new `jsPsych` trial in between the `Welcome` and `End` trials:

```js
// MyFirstScene trial
let myFirstSceneTrial = new PsyanimJsPsychTrial(MyFirstScene, MyFirstScene.key);

timeline.push(myFirstSceneTrial.jsPsychTrialDefinition);
```

Next, `using psyanim-cli` let's create a `MyFirstMovementComponent` to add to an `entity` in our `scene`, which will move the `entity` back and forth on the screen:

```bash
psyanim --component MyFirstMovementComponent
```

Open up `MyFirstMovementComponent.js` under the `/src` directory and replace it's contents with the following:

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

Open up `MyFirstScene.js` under your `/src` directory and replace it's contents with the following:

```js
import { PsyanimConstants } from 'psyanim2';

import MyFirstMovementComponent from './MyFirstMovementComponent.js';

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

In a terminal, start a watch service to watch your code for changes with:

```bash
npm run watch
```

In a separate terminal, start up a local server to host your code on port 3000 with:

```bash
npm run serve
```

Navigate to `localhost:3000` in your web browser and you should be able to play through your experiment, with the `MyFirstScene` trial showing the following:

<p align="center">
  <img src="./imgs/getting_started_final_result.gif" />
</p>

## 4. Scenes and Entities

## 5. Entities and Components

## 6. Entity Prefabs

