# Psyanim 2.0: Entity Prefabs

You can check out the full scene definition for this tutorial [here](TODO).

## 1. Introduction to `Entity Prefabs`

In the last tutorial on `Entities and Components`, we created a scene with an `Arrive Agent` that follows your mouse cursor around by adding the relevant `components` to an `entity`.

This was not too bad for something as simple as an `Arrive Agent` using mostly the default `parameter values` for each `component`, but some of the more complex agents have even more `components` and `parameters` for each component.

Moreover, we saw in the previous tutorial that some `components` will reference other `entities` or other `components` on the same `entity`.

This creates a sort of relationship between the `components` on an `entity` whenever you setup that type of `agent`.

We also saw that some `components` will have `parameter values` that are overriden by other `components` that reference it, often times so the user of a `behavior` or `agent component` does not need to configure the same `parameter` in multiple places where it is needed.

Instead, the user can configure a `parameter` (such as `maxSpeed`) in one place, maybe a higher-level component, and it will get propagated to the other `components` it references automagically.

To understand this for each `component` type, it is necessary to look at the `API documentation` for that `component`.

It can be cumbersome and error prone to have to capture this relationship between the `components` everytime you setup an `agent`.

This is where the concept of `Entity Prefabs` comes into play in `Psyanim 2.0`.

`Entity Prefabs` provide a much easier, reusable way to create and configure `entities` with specific sets of `components`.

In order to encapsulate these relationships between components on the same entity, especially for a reusable AI `behavior` or `agent`, multiple components are sometimes packaged up into an `Entity Prefab`.

The `Entity Prefab` encapsulates one or more `components` necessary to create an `agent` with specific `state` and `behaviors`, while also providing a single unified interface to (an often simpler) set of parameters without requiring the user to know anything about all the interfaces to each of the individual components its composed of.

## 2. Rebuilding our `MyArriveAgent` scene with prefabs

To see this `Entity Prefab` concept in action and better understand it's purpose, let's rebuild the `MyArriveAgent` scene again (from previous [tutorial](/overview/entities_and_components.md)), but this time using `prefabs` instead of adding / configuring every single component manually.

Remember that this is a smaller, simpler contrived example, and some prefabs, such as the `Playfight Agent Prefab` or the `Predator/Prey Agent Prefabs` will be even more complex, thus the `prefab` is even more helpful there.

Let's start by creating a new scene to work in.  In your terminal, navigate to your project directory and run the command:

```bash
npx psyanim-cli --scene MyArriveAgentPrefabScene
```

As in the previous tutorials, add this scene to a new `jsPsych trial` in your `index.js` and be sure to add it to your timeline (details omitted here for a chance to practice yourself :).

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

This looks just like our `MyArriveAgent` scene from the `Entities and Components` tutorial, except we haven't added any components to this agent.

## 3. Instantiating a `prefab`

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

Reload your scene in the browser and you should now see that this `MyArriveAgentPrefabScene` behaves exactly like our original `MyArriveScene` from the previous tutorial, only all the necessary components were added for us by the `prefab`!

## 4. Summary of `Entity Prefabs`

In this tutorial, we saw how `Entity Prefabs` allow us to reuse existing combinations of `components` to instantiate `entities` in different `scenes` without having to wire up and configure all the `components` manually every time, and they will also often expose a simpler set of `parameters` via the `prefab` interface.

`Psyanim 2.0` has a growing library of `prefabs` you can leverage in your experiments, so check out the [API docs](https://github.com/thefinnlab/psyanim-api-docs) to see what's there.