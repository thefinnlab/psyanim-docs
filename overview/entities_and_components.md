# Psyanim 2.0: Entities and Components

You can check out the full scene definition for this tutorial [here](https://github.com/thefinnlab/hello-psyanim2-tutorial/blob/master/src/MyArriveScene.js).

## 1. A deeper dive into `entities` and `components`

As we've seen in the previous tutorials, `entities` are things that live in `scenes` and we can give different `entities` different `state` and `behavior` by attaching `components` to them.

We'll explore this in greater detail by putting together a simple `scene` where an `agent` carries out an `Arrive Behavior` with a `target` that's following the `mouse cursor` around.

To visualize where the `agent`'s target is, we'll add an entity to the canvas that follows the `mouse cursor` and give it a small circular shape.

To make this entity follow the mouse cursor, we'll attach a `PsyanimMouseFollowTarget` component to it.  This component will make any entity follow the `mouse cursor`.

Then, we'll create another `entity` for our `Arrive Agent`.  This `agent` will then follow the first entity we created using a classic `Arrive Behavior`, which happens to be following the `mouse cursor`.

This tutorial will assume you've already setup your `experiment project` and completed the previous tutorials leading up to this point.

## 2. Creating our scene and adding two entities

The first thing we'll do is add two `entities` to our `scene`: one that follows the `mouse cursor` and one that follows the first `entity`.

To do this, let's navigate to our project in a terminal and run the following command to create our new scene:

```bash
npx psyanim-cli -s MyArriveScene
```

Next, let's open up our index.js and add this scene to a jsPsych trial by adding the following import statement at the top and declaring the trial object below:

```js
...
import MyArriveScene from './MyArriveScene';
...
let arriveSceneTrial = {
    type: PsyanimJsPsychPlugin,
    sceneKey: MyArriveScene.key,
    sceneParameters: { }
};
...
```

We also need to add our `arriveSceneTrial` to the timeline we pass to `jsPsych.run()`:

```js
jsPsych.run([welcome, myFirstSceneTrial, interactiveEvadeAgentTrial, arriveSceneTrial, goodbye]);
```

Now that we have our scene all setup, let's add the two entities to our entities array in `MyArriveScene.js`:

```js

import { PsyanimConstants } from 'psyanim2';

export default {

    key: 'MyArriveScene',
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
};
```

If you reload your scene in the browser now,  you'll see the two entities in the scene, but not moving at all.

The `mouseFollowTarget` entity above is the circle shaped object that will follow the mouse cursor around.

The `agent1` entity above is the agent entity that will follow the `mouseFollowTarget` around using an `Arrive Behavior`.

Notice that we added these two entities simply by adding two objects to the array.

Each `entity` needs a `name` property, which uniquely identifies them in the scene.  This is the only required field for an entity.

The `initialPosition` field is optional, but can allow you to place the entity at some starting position in the scene.  If not supplied, the entity is placed at the `world origin` (0, 0).

The `shapeParams` field is an object which supplies important shape information about the entity, e.g. shape, color, dimensions.  If not supplied, the entity will not have a visual representation in the scene.

## 3. Adding `mouse follow` behavior to our `mouseFollowTarget`

To make these entities move, we'll need to add some components that contain logic for moving them.

Let's start by updating our `psyanim2` import statement at the top of the file to include the components we want to add to our entities:

```js
import { 
    PsyanimConstants,
    PsyanimMouseFollowTarget,
    PsyanimVehicle,
    PsyanimArriveBehavior,
    PsyanimArriveAgent,
} from 'psyanim2';
```

The next step is adding a `PsyanimMouseFollowTarget` component to the `components` array to our `mouseFollowTarget` as follows:

```js
...
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
        }
...
```

Reload your scene in the browser and you should see the circular-shaped `mouseFollowTarget` following your mouse cursor around in the scene!

That's all there is to it!

You can add a `PsyanimMouseFollowTarget` to any `entity` in your scene definition's `entities` array and you will automagically get this cursor-following behavior!

Each `component` you add in the entity's `components` array contains only one required field, the component `type`.  This is `class type` of the `component`.

Components can often have parameters associated with them, which would be modified via the `params` property of the `component` object.

In this case, we do not have any parameters to configure for our `PsyanimMouseFollowTarget` component, so we can omit the `params` property of the `component`.

## 4. Adding components with parameters & dependencies

What we have so far is great, but our triangle-shaped `agent1` entity still isn't doing anything.

To make this `entity` carry out an `Arrive Behavior`, we'll need to add 3 different `components`:

- `PsyanimVehicle`
- `PsyanimArriveBehavior`
- `PsyanimArriveAgent`

The `PsyanimVehicle` component carries out the actual application of physics forces or velocities to our entity.  It is fundamental to all steering behaviors.

The `PsyanimArriveBehavior` component contains all the necessary logic for computing steering forces at each simulation frame, as well as the necessary parameters which make this behavior configurable.

The `PsyanimArriveAgent` component encapsulates the `PsyanimVehicle` and `PsyanimArriveBehavior` and a `target` entity to follow, while also maintaining the responsibility of querying the `PsyanimArriveBehavior` each frame for the steering forces and applying them to the `PsyanimVehicle`.

The separation of concerns between the `PsyanimArriveBehavior` and the `PsyanimArriveAgent` allows us to reuse the `PsyanimArriveBehavior` on other agents which have more complex state machines composed of multiple behaviors.

This steering architecture will be discussed in greater detail in the `AI Steering Behaviors` tutorial.

So, let's get to adding these components one at a time, starting with the `PsyanimVehicle` component.  Update your `agent1` entity definition to have a `components` field that looks like the following:

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
            components: [
                {
                    type: PsyanimVehicle,
                }
            ]
        }
...
```

Here, there's only 1 object in our `components` array for the `agent1` entity: our `PsyanimVehicle` component definition.

Similar to the `PsyanimMouseFollowTarget` component, this `component` only requires a `type` field.

The `PsyanimVehicle` does have some `configurable parameters`, but we're OK to leave them as the defaults, omitting the `params` field, since the `PsyanimArriveBehavior` component will override them with the settings we supply it anyhow.

Now, let's add the next component, the `PsyanimArriveBehavior` as follows:

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
            ]
        }
...
```

Adding this behavior is as simple as adding a second object to the `components` array of our `agent1` entity.

Notice this `component definition` contains a `params` field, though, unlike the last two components we worked with.

This is because the `PsyanimArriveBehavior` has 3 parameters for which we want to override the default values: 

- `maxSpeed`
- `innerDecelerationRadius`
- `outerDecelerationRadius`

The details of what these fields are for is not very important here, so we will gloss over it.

What is important to note, however, is that these fields are all javascript `number` types.  So, we must take care to supply the correct `type` of value for each field in the `params` object.

Finally, let's add our `PsyanimArriveAgent` component, including it's `params` that we are going to set:

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
        }
...
```

This third `component` follows the same pattern as all of the other ones.

It has a mandatory `type` field which contains the actual component type.

However, the `PsyanimArriveAgent` has more complex values for its component parameters.

This is because the `PsyanimArriveAgent` has parameters which are references to other entities and components.

For example, the `PsyanimArriveAgent`'s first component parameter is the `arriveBehavior` field.

This field should be supplied with a reference to the `PsyanimArriveBehavior` component for this agent.

The way we supply a reference to a component on a particular agent is to supply an object with two fields: `entityName` and `componentType`.

The first field, `entityName` describes the entity which we want to reference a component on.  Since `entity` names are always unique within a scene, this field uniquely identifies a particular entity.

The second field, `componentType` describes the `type` of the `component` we want to reference on said `entity`.  Since each entity can only have 1 `component` of a particular `type` attached, this uniquely identifies which component instance we are referring to on this entity.

Note that, if we want to reference an `entity`, and not a `component`, we simply omit the `componentType` field.  This will make the reference to the `entity` itself, not any particular component attached to it.

Applying this logic to the `vehicle` parameter of our `PsyanimArriveAgent`, we can see this parameter receives a reference to the `PsyanimVehicle` component of the `agent1` entity.

Lastly, the `target` of the `PsyanimArriveAgent` is an entity reference (notice the `componentType` field is omitted here).

This `target` field specifies the `entity` we want this `PsyanimArriveAgent` to follow, which is the `mouseFollowTarget` entity we created earlier.

Reload your scene in the browser and you should not see the triangle-shaped `agent` entity following your `mouseFollowTarget` as you move your mouse around!

Great work - you've learned how to add `entities` and `components` to a scene in general in Psyanim 2.0!