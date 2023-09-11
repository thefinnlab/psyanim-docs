# Psyanim 2.0: Scenes and Entities

Note: Full tutorial project + source code can be found [here](https://github.com/thefinnlab/psyanim-overview-tutorials/tree/scenes-and-entities).

## 1. The Psyanim Scene and Psyanim Entity

A `Psyanim Scene` is an abstraction for the 2D world we are simulating.

A `Psyanim Entity` is an abstraction for anything that exists in a Psyanim Scene with a particular location, rotation, and (optionally) a visual representation which can have physics applied to it.

The `PsyanimScene` acts as a container for `PsyanimEntity` objects.

Any object in your simulation that needs a position or orientation in the scene, a visual representation in the scene, or needs to have physics applied to it should be added to the scene as an entity.

To get a more hands-on feel for this, let's put together a simple scene where the player controls a circle-shaped agent and a triangle-shaped AI agent is trying to evade him/her.

## 2. Project and scene setup

For this tutorial, we will continue where we left off in the previous [`Hello Psyanim 2.0`](/overview/hello_psyanim_2.md) tutorial.

Let's go ahead and delete `EmptyScene.js` under the `./src` directory and open `index.js`.

Delete the `import EmptyScene...` line at the top and the line where we call:

```js
PsyanimApp.Instance.config.registerScene(EmptyScene);
```

Then, remove the following lines where we create the `emptySceneTrial` object in `jsPsych`:

```js
...

let emptySceneTrial = {
    type: PsyanimJsPsychPlugin,
    sceneKey: EmptyScene.key,
};

...
```

Next, update the `jsPsych.run()` call so it no longer has the `emptySceneTrial` in its timeline, like so:

```js
...

jsPsych.run([welcome, myFirstSceneTrial, goodbye]);
```

Now it's time to create our new `InteractiveEvadeAgent` scene using Psyanim-CLI.

In your terminal, at the root of the project directory, run the following command to create your scene:

```bash
npx psyanim-cli --scene InteractiveEvadeAgent
```

You should see `InteractiveEvadeAgent.js` show up in your `./src` directory.

The contents of this file should look like the following:

```js
import { PsyanimConstants } from 'psyanim2';

export default {
    key: 'InteractiveEvadeAgent',
    wrapScreenBoundary: false,
    entities: [],
    navgrid: {
        cellSize: 10,
        obstacles: []
    }
}
```

This is a basic `scene definition` in `Psyanim 2.0`!  Let's break this down a bit here before moving forward.

Every `scene` in `Psyanim 2.0` is composed of `entities` which, as we mentioned in the previous section, are just 'things' that live in the scene.

Moreover, every `scene definition` in `Psyanim 2.0` must have a `key` which uniquely identifies this scene amongst the rest of the scenes in the project.

Certain functionality in `Psyanim 2.0`, such as `Point-Click Movement With Obstacles`, uses `2D Pathfinding` to function, which in turn requires a `navigation grid` to be defined in the scene, since the pathfinder operates in this grid-space, not world-space.

Since this tutorial won't be using any `2D Pathfinding` features, we can safely remove the navigation grid field from our scene definition.

The `wrapScreenBoundary` field is used to control whether or not the agent can leave the screen boundary on one side, appearing on the other as if the 2D cavnas is a 3D object that's been 'unwrapped' into screen space.

Since we want this screen boundary wrapping behavior for this experiment trial, let's set the `wrapScreenBoundary` field in our `scene definition` to `true`.

Let's open up our `index.js` and add our newly created scene to our `jsPsych` experiment timeline as a trial.

Add an import statement at the top of the file to bring in our `InteractiveEvadeAgent` scene definition:

```js
import InteractiveEvadeAgent from './InteractiveEvadeAgent';
```

Next, we must register this scene with `PsyanimApp` before it can be used in a `jsPsych` trial, by adding the following line before we call `PsyanimApp.Instance.run()`:

```js
PsyanimApp.Instance.config.registerScene(InteractiveEvadeAgent);
```

Now it's time to create a `jsPsych` trial that uses this Psyanim `scene definition`.  Add the following code after `myFirstSceneTrial` is declared:

```js
let interactiveEvadeAgentTrial = {
    type: PsyanimJsPsychPlugin,
    sceneKey: InteractiveEvadeAgent.key,
};
```

Finally, we just need to add this to our timeline by updating our `jsPsych.run()` call to include the new trial object we created:

```js
jsPsych.run([welcome, myFirstSceneTrial, interactiveEvadeAgentTrial, goodbye]);
```

Great work - our project is all setup and we're ready to start building out our scene!

## 3. Adding a 'player' entity to our scene

The time has finally come to add our first `entity` to our `InteractiveEvadeAgent` scene!

Let's open `InteractiveEvadeAgent.js` and modify the `psyanim2` import statement so it imports the necessary classes as follows:

```js
import 
{ 
    PsyanimConstants,
    PsyanimPlayerController,
    PsyanimEvadeAgentPrefab,
    PsyanimEvadeAgent
    
} from 'psyanim2';
```

At this point, your `scene definition` object should look like the following:

```js
export default {
    key: 'InteractiveEvadeAgent',
    wrapScreenBoundary: true,
    entities: [],
}
```

In order to add an entity to the scene for the player, all we need to do is add a javascript object to our entities array like so:

```js
export default {
    key: 'InteractiveEvadeAgent',
    wrapScreenBoundary: true,
    entities: [
        {
            name: 'player',
            initialPosition: { x: 400, y: 300 }
        }
    ],
}
```

The `name` field is a unique ID for the `entity` in our scene.  Take care to ensure no two entities have the same name in the scene or Psyanim might yell at you :D

The `initialPosition` field is pretty straight forward - it is the position that the entity assumes at simulation time `t = 0` (the very first rendered frame).

We have set the `initialPosition` to `(400, 300)`, the center of our canvas.  By default, the canvas size in `Psyanim 2.0` is `800 x 600` pixels.

If you reload your experiment in the browser after adding these lines for our entity, you'll notice you have a faint yellow circle in the middle of the scene.

This is great, but I kind of wanted our player to be a `blue circle`, and with a radius that's a bit more noticeable.

This is where the `shapeParams` field of an `entity` comes in handy.  Let's update our `entity` object so it has the following `shapeParams`:

```js
export default {
    key: 'InteractiveEvadeAgent',
    wrapScreenBoundary: true,
    entities: [
        {
            name: 'player',
            initialPosition: { x: 400, y: 300 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
                radius: 12,
                color: 0x0000ff
            },            
        }
    ],
}
```

The `shapeType` field of our `shapeParams` object controls the shape type. Currently, Psyanim 2.0 supports `Circles`, `Triangles`, and `Rectangles`.

The `radius` field is only used for `circles`, and it's given in `pixels (px)`.

The `color` field is a standard RGB hex color code.

Reload your experiment in the browser and you should see a `blue circle` sitting in the center of your canvas!

This `blue circle` sure does look wonderful, but it's supposed to be a player-controlled `entity`!

How can we make this thing move in response to player control input?

The answer is simple: add a `PsyanimComponent` to it!

We'll go into greater depth about `PsyanimComponents` in the next tutorial section, but for now, just know that `components` provide us a way to add specialized state and behaviors to any entity in any scene, such as making an `entity` move in response to keypresses or having the `entity` respond to other `entities` in the environment.

For player keyboard movement, Psyanim 2.0 comes with a WASD keyboard `PsyanimPlayerController` component that we can use.

Let's add this to the `components` array field of our `entity` object as follows:

```js
export default {
    key: 'InteractiveEvadeAgent',
    wrapScreenBoundary: true,
    entities: [
        {
            name: 'player',
            initialPosition: { x: 400, y: 300 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
                radius: 12,
                color: 0x0000ff
            },
            components: [
                { type: PsyanimPlayerController }
            ]      
        }
    ],
}
```

Each `component` of an `entity` is added as a Javascript object to our `components` array.

In the above snippet, we added the `PsyanimPlayerController` component to our `entity` by adding a Javascript object to it's `components` array, supplying the `component type` using the `type` field.

Great work - if you reload your experiment in the browser and navigate to the trial we setup, you should be able to move your `blue circle` entity around in the scene by pressing the `W`, `A`, `S`, or `D` keys on your keyboard!

## 4. Adding an AI-controlled entity to our scene

Now all we need to do is add an `AI-controlled entity` to our scene.  We often refer to AI-controlled entities as `agents`.

Let's start by adding a second `entity` definition to our `scene definition`'s `entities` array with the following `initialPosition` and `shapeParams`:

```js
export default {
    key: 'InteractiveEvadeAgent',
    entities: [
        {
            name: 'player',
            initialPosition: { x: 400, y: 300 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
                radius: 12,
                color: 0x0000ff
            },
            components: [
                { type: PsyanimPlayerController }
            ]
        },
        {
            name: 'agent1',
            initialPosition: { x: 600, y: 450 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
                base: 16, altitude: 32, color: 0x00ff00         
            }
        },
    ]
};
```

This `agent` has a green triangle shape and we place it in the lower-left quadrant of the `scene canvas`.

Adding an `evade behavior` to an `agent` involves adding 4 different components to it and configuring each one of them properly: 

`PsyanimVehicle`, `PsyanimFleeBehavior`, `PsyanimEvadeBehavior`, and `PsyanimEvadeAgent`

While this might not be a large number of components, you'd have to be familiar with each one individually and understand how to use them together.

Lucky for us, there's an easier way to add all the necessary components and configure them all properly: `Entity Prefabs`!

`Entity Prefabs` are like a blueprint for creating a certain type of agent, one with potentially complex behaviors produced by combining different components together in various ways.

Don't worry too much about all the details for now, but just know that, for a given behavior, an `Entity Prefab` will add all the necessary components to an agent for you and configure them properly.

The `prefab` should also expose only the necessary configuration parameters you will need to care about, making configuration as simple as possible.

To get our `agent` behaving like a `PsyanimEvadeAgent`, let's add the `PsyanimEvadeAgentPrefab` to our new entity definition like so:

```js
...
        {
            name: 'agent1',
            initialPosition: { x: 600, y: 450 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
                base: 16, altitude: 32, color: 0x00ff00         
            },
            prefab: { type: PsyanimEvadeAgentPrefab },
        }
...
```

The `PsyanimEvadeAgentPrefab` adds a `PsyanimEvadeAgent` `component` to our `entity`, which requires that we specify a target for our `agent` to evade.

To do this, we can use the `components` array of our `entity definition` to specify another entity in the scene as the `evade agent`'s target, referencing it's by name as follows:

```js
        {
            name: 'agent1',
            initialPosition: { x: 600, y: 450 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
                base: 16, altitude: 32, color: 0x00ff00         
            },
            prefab: { type: PsyanimEvadeAgentPrefab },
            components: [
                {
                    type: PsyanimEvadeAgent,
                    params: {
                        target: {
                            entityName: 'player'
                        }
                    }
                }
            ]
        }
```

Here, we set the `target` parameter of the `PsyanimEvadeAgent` component to the `player` entity.

Don't worry too much about `prefabs` and `components` for now, we'll explore them in more detail in the next tutorials.

At this point, your code should look like [this](https://github.com/thefinnlab/psyanim-overview-tutorials/blob/scenes-and-entities/src/InteractiveEvadeAgent.js).

---

Go ahead and reload your experiment in the browser and you should see your blue, circle-shaped `player` entity and the green, triangle-shaped `evade agent` we just created.

When you try to approach the `evade agent` using the WASD keys on your keyboard, the `agent` should move away from where it predicts you're headed.

Great work - you've just setup an interactive scene for your experiment with a keyboard-controlled `player` entity and an `AI agent` that evades the player!