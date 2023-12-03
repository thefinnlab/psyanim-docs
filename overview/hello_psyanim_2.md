# <ins>Getting Started with Psyanim 2.0</ins>

***Note: Full tutorial project + source code can be found [here](https://github.com/thefinnlab/hello-psyanim2).***

## 1. Core concepts

Psyanim 2.0 is a framework for creating browser-based psychology / cognitive science research experiments involving 2D procedural animation.

The framework is built on top of [Phaser 3](https://phaser.io/) and has a flexible [component architecture](https://gameprogrammingpatterns.com/component.html).

---

In Psyanim 2.0, everything that is seen in the world lives in a `PsyanimScene`, which is an abstraction for the 2D world we are simulating.

The `PsyanimScene` acts as a container for `PsyanimEntity` objects.

---

Any object that exists in a scene, regardless of its visual representation, is called a `PsyanimEntity`.

A `PsyanimEntity` acts a container for `PsyanimComponents`.  

More technically, it's an abstraction for anything that exists in a Psyanim Scene with a particular location, rotation, and (optionally) a visual representation which can have physics applied to it.

Entities may or may not have a visual representation in the scene.

Moreover, entities alone do not have any logic or behaviors.  While entities have no user-defined state, they do have a position, orientation and velocity in the world, and can have forces / accelerations applied to them.

Any object in your simulation that needs a position or orientation, a visual representation in the scene, or needs to have physics applied to it, should be added to the scene as an entity.

---

All user-defined state and behaviors are encapsulated in `PsyanimComponents`, which are reusable scripts that can be attached to a `PsyanimEntity` and offer hooks into the real-time update loop of each scene.

---

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

---

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

To add an `entity` in the `scene definition` above, we add a single `entity definition` to our `entities` array called `agent1`.

The `agent1` entity has a red circle representation in the scene and is centered at pixel coordinates `(400, 300)` in the scene.

Notice we add our `MyFirstMovementComponent` to our `agent1` entity by adding a single object to its `components` array property.  The `type` field of the `component definition` object specifies the class type of the `component` we want to add.

---

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

Let's create another scene to get more familiar with `Psyanim Scenes` and `Psyanim Entities`.

We'll create a scene with a player-controlled agent as well as an agent controlled by AI (artificial intelligence).

In your terminal, run the following command to create your new scene:

```bash
psyanim -s InteractiveEvadeAgent
```

You should see `InteractiveEvadeAgent.js` show up in your `/src` directory.  Open this file up and you should see the following:

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

This is a basic `scene definition` in `Psyanim 2.0`.

Every `scene` in `Psyanim 2.0` is composed of `entities` which, as we mentioned in the previous section, are just 'things' that live in the scene.

Moreover, every `scene definition` in Psyanim 2.0 must have a `key` which uniquely identifies this scene amongst the rest of the scenes in the project.

The `wrapScreenBoundary` field is used to control whether or not the agent can leave the screen boundary on one side, appearing on the other as if the 2D canvas is a 3D object that's been 'unwrapped' into screen space.

The `navgrid` field is only needed for things which require a `navigation grid`, such as `AI pathfinding`.

Since this tutorial won't be using any `2D Pathfinding` features, we can safely remove the `navgrid` field from our scene definition.

---

Let's add a `player` entity to our scene by first importing the appropriate `components` at the top of our `scene definition` file:

```js
import { 
    PsyanimConstants,
    PsyanimPlayerController,

    PsyanimEvadeAgentPrefab,
    PsyanimEvadeAgent

} from 'psyanim2';
```

Let's also set the `wrapScreenBoundary` field to `true` to allow players to move around the world without running into any boundaries:

```js
...
wrapScreenBoundary: true,
...
```

Next, add the following javascript object to your `entities` array of the `scene definition`:

```js
...
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
...
```

The `name` field is a unique ID for the `entity` in our `scene`. Take care to ensure no two entities have the same name in the scene.

The `initialPosition` field declares the position that the entity assumes at simulation time t = 0 (the very first rendered frame), unless a `component` overrides this.

We have set the initialPosition to (400, 300), the center of our canvas. By default, the canvas size in Psyanim 2.0 is 800 x 600 pixels.

---

The `shapeParams` field allows us to define a `shape`, `size` and `color` for the `entity`.

The `shapeType` field of our shapeParams object controls the entity's shape. Currently, Psyanim 2.0 supports `Circles`, `Triangles`, and `Rectangles`.

The `radius` field is only used for circles, and it's given in pixels (px).

The `color` field is a standard RGB hex color code.

---

The `components` array allows us to attach `Psyanim Components` to our `entity`.  

Recall that `entities` themselves do not contain custom state or behavior.  

A `Psyanim Component` is just a script that contains custom state and behavior.

Here, the `player` entity has a `PsyanimPlayerController` component that contains logic for moving the player in response to keyboard keypresses.

---

Now, let's create our `AI agent` by adding the following `entity definition` to the `entities` field of our `scene definition`:

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
...
```

The AI agent's `entity definition` is not much different from that of the player entity.

The only new concept we see here is that the `PsyanimEvadeAgent` component has some parameters that need to be supplied in order to work correctly.

Every `PsyanimComponent` attached to an `entity` can be supplied a `params` field which is a javascript object containing any parameters that we would like to set or override.

For the `PsyanimEvadeAgent`, we set it's `target` parameter to the player so it will attempt to avoid the player entity.

---

The final step is to add this scene to a new trial in our `index.js`.  This is left as an exercise here, but you can verify how to do this in the previous section or by looking at the [example repo](https://github.com/thefinnlab/hello-psyanim2/blob/master/src/index.js).

Reload your experiment in the browser and you should be able to move the player entity around with the `W`, `A`, `S`, and `D` keys on your keyboard.

Great work - you should see the evade agent avoiding the player entity as you move around your scene!

## 5. Entities and Components

To dive a bit deeper into how to configure `components` in `entity definitions` within our `scene definition`, let's put together another `scene` where an agent follows our mouse cursor around during the `trial`.

The first thing we'll do is add two `entities` to our scene: one that follows the `mouse cursor` and one that follows the the first `entity`.

To do this, let's navigate to our project in a terminal and run the following command to create our new scene:

```bash
psyanim -s MyArriveScene
```

Be sure to add this scene to a new trial in your `index.js`.

Then, open up the scene we just created and let's add two entities to it as follows:

```js
...
entities: [
    {
        name: 'mouseFollowTarget',
        initialPosition: { x: 400, y: 300 },
        shapeParams: {
            shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
            radius: 4,
            color: 0x00ff00
        }
    },
    {
        name: 'agent1',
        initialPosition: { x: 600, y: 450 },
        shapeParams: {
            shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
            base: 16, altitude: 32, 
            color: 0xffc0cb            
        }
    }
]
...
```

We have our two entities in the scene now, but they don't have any `components` with movement logic.  Thus, they do not move at all.

To make these entities move, let's import some `components` at the top of our `scene definition` file:

```js
import { 
    PsyanimConstants,
    PsyanimMouseFollowTarget,
    PsyanimVehicle,
    PsyanimArriveBehavior,
    PsyanimArriveAgent,
} from 'psyanim2';
```

Next, let's add a `PsyanimMouseFollowTarget` component to our `mouseFollowTarget` entity so it follows our mouse cursor around the canvas by updating its `components` array as follows:

```js
...
components: [
    { type: PsyanimMouseFollowTarget }
]
...
```

You can add a `PsyanimMouseFollowTarget` to any entity in your `scene definition`'s `entities` array and you will automagically get this cursor-following behavior!

Each `component` you add in the entity's `components array` contains only one required field, the component `type`. This is the class type of the component.

`Components` can often have `parameters` associated with them, which would be modified via the `params` property of the component object.

In this case, we do not have any `parameters` to configure for our `PsyanimMouseFollowTarget` component, so we can omit the params property of the component.

Reload your experiment in the browser and you should see the small green circle `mouseFollowTarget` entity following your mouse cursor around.

---

Now, let's get our `agent` entity chasing the `mouseFollowTarget` entity around by updating it's entity definition.

To make this `entity` carry out an `Arrive Behavior`, we'll need to add 3 different `components`:

- `PsyanimVehicle`
- `PsyanimArriveBehavior`
- `PsyanimArriveAgent`

The `PsyanimVehicle` component carries out the actual application of physics forces or velocities to our entity.  It is fundamental to all steering behaviors.

The `PsyanimArriveBehavior` component contains all the necessary logic for computing steering forces at each simulation frame, as well as the necessary parameters which make this behavior configurable.

The `PsyanimArriveAgent` component encapsulates the `PsyanimVehicle` and `PsyanimArriveBehavior` and a `target` entity to follow, while also maintaining the responsibility of querying the `PsyanimArriveBehavior` each frame for the steering forces and applying them to the `PsyanimVehicle`.

The separation of concerns between the `PsyanimArriveBehavior` and the `PsyanimArriveAgent` allows us to reuse the `PsyanimArriveBehavior` on other agents which have more complex state machines composed of multiple behaviors.

This steering architecture will be discussed in greater detail in a later tutorial.

So, let's add these components one at a time, starting with the `PsyanimVehicle` component.

Update your `agent1` entity definition `components` array to look like the following:

```js
...
components: [
    {
        type: PsyanimVehicle,
    },
    {
        type: PsyanimArriveBehavior,
        params: {
            maxSpeed: 8,
            innerDecelerationRadius: 25,
            outerDecelerationRadius: 140
        }
    },
    {
        type: PsyanimArriveAgent,
        params: {
            arriveBehavior: {
                entityName: 'agent1',
                componentType: PsyanimArriveBehavior
            },
            vehicle: {
                entityName: 'agent1',
                componentType: PsyanimVehicle
            },
            target: {
                entityName: 'mouseFollowTarget'
            }
        }
    }
]
...
```

Similar to the `PsyanimMouseFollowTarget` component, the `PsyanimVehicle` component only requires a `type` field.

The `PsyanimVehicle` does have some configurable `parameters`, but we're OK to leave them as the defaults, omitting the params field, since the `PsyanimArriveBehavior` component will override them with the settings we supply it anyhow.

---

Notice the `PsyanimArriveBehavior` `component definition` contains a `params` field, unlike the last two components we worked with.

This is because the `PsyanimArriveBehavior` has 3 parameters for which we want to override the default values: 

- `maxSpeed`
- `innerDecelerationRadius`
- `outerDecelerationRadius`

The details of what these fields are for is not very important here, so we will gloss over it.

What is important to note, however, is that these fields are all javascript `number` types.  So, we must take care to supply the correct type of value for each field in the `params` object.

---

This third component, the `PsyanimArriveAgent`, follows the same pattern as all of the other ones.

It has a mandatory `type` field which contains the actual component type.

However, the `PsyanimArriveAgent` has more complex values for its component parameters.

This is because the `PsyanimArriveAgent` has parameters which are references to other entities and components.

For example, the `PsyanimArriveAgent`'s first component parameter is the `arriveBehavior` field.

This field should be supplied with a reference to the `PsyanimArriveBehavior` component for this agent.

---

The way we supply a reference to a component on a particular agent is to supply an object with two fields: `entityName` and `componentType`.

The first field, `entityName` describes the entity which we want to reference a component on.  Since `entity` names are always unique within a scene, this field uniquely identifies a particular entity.

The second field, `componentType` describes the `type` of the `component` we want to reference on said `entity`.  Since each entity can only have 1 `component` of a particular `type` attached, this uniquely identifies which component instance we are referring to on this entity.

Note that, if we want to reference an `entity`, and not a `component`, we simply omit the `componentType` field.  This will make the reference to the `entity` itself, not any particular component attached to it.

---

Applying this logic to the `vehicle` parameter of our `PsyanimArriveAgent`, we can see this parameter receives a reference to the `PsyanimVehicle` component of the `agent1` entity.

Lastly, the `target` of the `PsyanimArriveAgent` is an entity reference (notice the `componentType` field is omitted here).

This `target` field specifies the `entity` we want this `PsyanimArriveAgent` to follow, which is the `mouseFollowTarget` entity we created earlier.

---

Great work!  Reload your scene in the browser - you should see the triangle-shaped `agent` entity following your `mouseFollowTarget` as you move your mouse around!

## 6. Entity Prefabs

In the last section on `Entities and Components`, we created a scene with an `Arrive Agent` that follows your mouse cursor around by adding the relevant `components` to an `entity`.

This was not too bad for something as simple as an `Arrive Agent` using mostly the default `parameter values` for each `component`, but some of the more complex agents have even more `components` and `parameters` for each component.

Moreover, we saw in the previous section that some `components` will reference other `entities` or other `components` on the same `entity`.

This creates a relationship between the `components` on different `entities` whenever you setup that type of `agent`.

We also saw that some `components` will have `parameter values` that are overriden by other `components` that reference it, often times so the user of a `behavior` or `agent component` does not need to configure the same `parameter` in multiple places where it is needed.

Instead, the user can configure a `parameter` (such as `maxSpeed`) in one place, maybe a higher-level component, and it will get propagated to the other `components` it references automagically.

To understand this for each `component` type, it is necessary to look at the `API documentation` or `example scenes` for that `component`.

It can be cumbersome and error prone to have to capture this relationship between the `components` everytime you setup an `agent`.

This is where the concept of `Entity Prefabs` comes into play in `Psyanim 2.0`.

---

`Entity Prefabs` provide a much easier, reusable way to create and configure `entities` with specific sets of `components`.

In order to encapsulate these relationships between components on the same entity, especially for a reusable AI `behavior` or `agent`, multiple components are sometimes packaged up into an `Entity Prefab`.

The `Entity Prefab` encapsulates one or more `components` necessary to create an `agent` with specific `state` and `behaviors`, while also providing a single unified interface to (an often simpler) set of parameters without requiring the user to know anything about all the interfaces to each of the individual components its composed of.

---

To see these `Entity Prefabs` in action, let's rebuild the scene from the previous section, but this time using `prefabs` instead of adding / configuring every single component manually.

Remember that this is a smaller, simpler contrived example, and some prefabs, such as the `Playfight Agent Prefab` or the `Predator/Prey Agent Prefabs` will be even more complex, thus the `prefab` is even more helpful there.

Let's start by creating a new scene to work in.  In your terminal, navigate to your project directory and run the command:

```bash
psyanim --scene MyArriveAgentPrefabScene
```

As in the previous sections, add this scene to a new `jsPsych trial` in your `index.js`.

Open up `MyArriveAgentPrefabScene` and replace its contents with the following code:

```js
import { 
    PsyanimConstants,
    PsyanimMouseFollowTarget,
    PsyanimArriveAgentPrefab,
    PsyanimArriveAgent
} from 'psyanim2';

export default {
    key: 'MyArriveAgentPrefabScene',
    entities: [
        {
            name: 'mouseFollowTarget',
            initialPosition: { x: 400, y: 300 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
                radius: 4,
                color: 0x00ff00
            },
            components: [
                { type: PsyanimMouseFollowTarget }
            ]
        },
        {
            name: 'agent1',
            initialPosition: { x: 600, y: 450 },
            shapeParams: {
                shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
                base: 16, altitude: 32, 
                color: 0xffc0cb            
            }
        }
    ]
}
```

This looks just like our `MyArriveAgent` scene from the `Entities and Components` section, except we haven't added any components to `agent1`.

---

Instead of adding `components` to this `agent`, we are going to `instantiate` a `prefab` that attaches all of the `components` to it for us.

The `prefab` itself will have an interface of configurable `parameters` and will handle configuring all of the `components` properly based on the `parameter` values we set for it, or else the default values the `prefab parameters` have.

While the `prefab parameters` can be of any type, the only restriction is that a `prefab parameter` **can not** reference any other `entity` or `component` in a specific `scene`.  In this case, we must override the `component` values for this `entity`.

If we have an `entity` which is instantiated from a `prefab`, as our `agent1` entity here will be, we can still override the individual `component` values after it's instantiated by adding `parameters` for those `components` to the `entity`'s `components` array.

For our `agent1` entity, we will add the prefab to it's `entity definition`, and then we will override the `target` entity reference of the `PsyanimArriveAgent` component.

Update your `agent` entity definition to look like the following:

```js
...
{
    name: 'agent1',
    initialPosition: { x: 600, y: 450 },
    shapeParams: {
        shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
        base: 16, altitude: 32, 
        color: 0xffc0cb            
    },
    prefab: {
        type: PsyanimArriveAgentPrefab,
    },
    components: [
        {
            type: PsyanimArriveAgent,
            params: {
                target: {
                    entityName: 'mouseFollowTarget'
                }
            }
        }
    ]
}
...
```

You can see that the prefab has a `type` field, where we supply the type of `prefab` we want to use to instantiate this `entity`.

The `prefab` can also have a `params` field, which works exactly like the `params` field of a `component`, with the exception that a `prefab` parameter can not reference another entity or component.

In order to set the `target` property of the `PsyanimArriveAgent` component that the `PsyanimArriveAgentPrefab` adds to our `entity`, we simply override it in the `components` array of the `entity`, similar to how we might add a `component` to a `non-prefab entity`.

The difference, in this case, is that the `PsyanimArriveAgent` `component definition` we added to our `components` array is not *adding* any `components` to the `entity`.  Rather, it is *overriding* the existing `component`'s properties.

Reload your scene in the browser and you should now see that this `MyArriveAgentPrefabScene` behaves exactly like our original `MyArriveScene` from the previous section, only all the necessary components were added for us by the `prefab`!

---

In this section, we saw how `Entity Prefabs` allow us to reuse existing combinations of `components` to instantiate `entities` in different `scenes` without having to wire up and configure all the `components` manually every time, and they will also often expose a simpler set of `parameters` via the `prefab` interface.

`Psyanim 2.0` has a growing library of `prefabs` you can leverage in your experiments, so check out the [API docs](https://github.com/thefinnlab/psyanim-api-docs) to see what's there.