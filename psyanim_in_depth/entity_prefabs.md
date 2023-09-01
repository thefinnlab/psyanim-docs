# <ins>Psyanim In-Depth: Entity Prefabs</ins>

## 1. What's an `Entity Prefab`?

**An `Entity Prefab` is like a blueprint that captures the necessary components for creating an entity with specific state and behaviors.**

In addition to capturing the necessary components, an `Entity Prefab` should handle the component configuration and can expose custom configuration parameters, providing a simpler interface for entity creation than the dealing with the individual components themselves.

We've seen, in the previous tutorials, that `entities` are objects that live in a `scene`.

We've also learned that we can add state and behavior to an `entity` by attaching `components` to it.

The simplest `entities` may have no `components` or maybe just 1 `component` attached to them.  For these `entities`, it's easy enough to create more of them simply by attaching the same type of `component` to another `entity` in the same `scene` or in other `scenes`.

However, as soon as we start creating & using agents with more complex behaviors requiring many components, such as the `Playfight`, `Predator`, or `Prey`, attaching all the necessary `components` and configuring them all properly can become unecessarily repetitive and cumbersome.

`Entity Prefabs` provide a means to hide this complexity and minimize the amount of repetitive setup code when adding agents composed of many components to your scene. 

## 2. Complex entities: Building a playfight behavior from components

**To understand the problem better, let's create an experiment with a simple `playfight scene` with two agents executing our `Playfight` algorithm.**

At it's core, the `Playfight` algorithm involves two agents wandering about the world randomly, and at specified intervals, they attack by charging at each other until they make contact, at which point they return to wandering until the next time to attack.

**a. Let's create a directory called `hello-entity-prefabs` and setup our experiment:**

Navigate to the `hello-entity-prefabs` directory and run the following commands to initialize our experiment:

```bash
npm init -y
npm install git+https://github.com/thefinnlab/psyanim-2.git git+https://github.com/thefinnlab/psyanim-cli.git
npx psyanim-cli --init
```

We can go ahead and delete the `EmptyScene.js` and then create our `PlayfightScene.js` with the following command:

```bash
npx psyanim-cli -s PlayfightScene
```

Open up `index.js` and replace its contents with the following:

```js
import { PsyanimApp } from 'psyanim2';

import PlayfightScene from './PlayfightScene';

/**
 *  Setup Psyanim and PsyanimJsPsychPlugin
 */
PsyanimApp.Instance.config.registerScene(PlayfightScene);

PsyanimApp.Instance.run();
```

**b. Let's implement our `PlayfightScene` by adding all the necessary entities and components:**

Open `PlayfightScene.js`, delete the `init()`, `preload()` and `update(t, dt)` methods, and then update the `psyanim2` import so it has the following classes:

```js
import 
{ 
    PsyanimScene,
    PsyanimConstants,
    PsyanimVehicle,
    PsyanimArriveBehavior,
    PsyanimAdvancedFleeBehavior,
    PsyanimSeekBehavior,
    PsyanimWanderBehavior,
    PsyanimPlayfightBehavior,
    PsyanimPlayfightAgent

} from 'psyanim2';
```

Next, add the following code to the end of the create() method to setup our first `Playfight` agent entity:

```js
    create() {

        super.create();
        
        this.screenBoundary.wrap = false;

        /**
         *  Setup agent 1
         */

        // create entity with appropriate shapeParams
        let agent1 = this.addEntity('agent1', 200, 300, {
            shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE, 
            radius: 12, color: 0xff0000
        });

        // the vehicle component is the fundamental movement component for steering behaviors
        let vehicle1 = agent1.addComponent(PsyanimVehicle);
    
        // the 'arrive' behavior is used for the 'charge' in playfight
        let arrive1 = agent1.addComponent(PsyanimArriveBehavior);
        arrive1.maxSpeed = 9;
        arrive1.maxAcceleration = 0.4;
        arrive1.innerDecelerationRadius = 12;
        arrive1.outerDecelerationRadius = 30;

        // flee allows us to keep the agents separated during wander
        let flee1 = agent1.addComponent(PsyanimAdvancedFleeBehavior);
        flee1.maxSpeed = 4;
        flee1.maxAcceleration = 0.2;
        flee1.panicDistance = 100;

        // wander uses 'seek' behavior to propel agent towards wander target
        let seek1 = agent1.addComponent(PsyanimSeekBehavior);
        seek1.maxSpeed = 4;
        seek1.maxAcceleration = 0.2;

        // add wander component, noting how it references the seek behavior
        let wander1 = agent1.addComponent(PsyanimWanderBehavior);
        wander1.seekBehavior = seek1;
        wander1.radius = 50;
        wander1.offset = 250;
        wander1.maxWanderAngleChangePerFrame = 20;

        // add playfight behavior component, with methods to compute steering
        let playfight1 = agent1.addComponent(PsyanimPlayfightBehavior);
        playfight1.breakDuration = 2000;
        playfight1.fleeBehavior = flee1;
        playfight1.arriveBehavior = arrive1;
        playfight1.wanderBehavior = wander1;
        
        // the playfight agent is necessary to actually steer the agent using the 'behavior' component
        let playfightAgent1 = agent1.addComponent(PsyanimPlayfightAgent);
        playfightAgent1.playfightBehavior = playfight1;
        playfightAgent1.vehicle = vehicle1;
    }
```

We will not go into great depth here about the `AI steering behaviors`, but we will at least touch a bit on the architecture to understand why there are so many components necessary for a `PsyanimPlayfightAgent`.

Notice the first `component` added to `agent1` is a `PsyanimVehicle`.  This is the fundamental class which applies steering forces to the `entity`.

Every steering behavior component is designed simply to compute a steering force.  It does not actually drive the entity's velocity or position.

The `Agent` components in our AI steering library are the ones that actually query the `Behavior` classes for the steering force each frame and then apply those forces to the `PsyanimVehicle`.

Each steering `Agent` may need to do this in a different way, thus we have different `Agent` types for different `Behavior` types.

For the `PsyanimPlayfightBehavior`, the `PsyanimPlayfightAgent` component is responsible for actually querying the `behavior` component for the steering force each frame and then applying it to the `PsyanimVehicle` component.

Moreover, the `Playfight` algorithm is a state machine that is composed of several other more fundamental steering behaviors, namely the `Arrive` behavior for charging towards other agents, the `Wander` behavior for wandering around non-deterministically, and the `Flee` behavior for maintaining distance from another entity.

All of these `components` must be added to the `entity` and *properly configured* for the `playfight algorithm` to work.

Why not have all of the code for these components in a single component?  

It's about <b>*modularity*</b> and <b>*code reuse*</b>.

Other algorithms, scenes or prefabs may use the `Flee` behavior component, or the `Arrive` behavior component, and we don't want devs to have to reimplement it every time we want an agent to carry out a `flee` or `arrive` behavior.

Let's setup the rest of our `PlayfightScene.js` by adding the second entity and setting their target parameters to play with each other.  Your `create()` method should look like the following:

```js
    create() {

        super.create();
        
        this.screenBoundary.wrap = false;

        /**
         *  Setup agent 1
         */

        // create entity with appropriate shapeParams
        let agent1 = this.addEntity('agent1', 200, 300, {
            shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE, 
            radius: 12, color: 0xff0000
        });

        // the vehicle component is the fundamental movement component for steering behaviors
        let vehicle1 = agent1.addComponent(PsyanimVehicle);
    
        // the 'arrive' behavior is used for the 'charge' in playfight
        let arrive1 = agent1.addComponent(PsyanimArriveBehavior);
        arrive1.maxSpeed = 9;
        arrive1.maxAcceleration = 0.4;
        arrive1.innerDecelerationRadius = 12;
        arrive1.outerDecelerationRadius = 30;

        // flee allows us to keep the agents separated during wander
        let flee1 = agent1.addComponent(PsyanimAdvancedFleeBehavior);
        flee1.maxSpeed = 4;
        flee1.maxAcceleration = 0.2;
        flee1.panicDistance = 100;

        // wander uses 'seek' behavior to propel agent towards wander target
        let seek1 = agent1.addComponent(PsyanimSeekBehavior);
        seek1.maxSpeed = 4;
        seek1.maxAcceleration = 0.2;

        // add wander component, noting how it references the seek behavior
        let wander1 = agent1.addComponent(PsyanimWanderBehavior);
        wander1.seekBehavior = seek1;
        wander1.radius = 50;
        wander1.offset = 250;
        wander1.maxWanderAngleChangePerFrame = 20;

        // add playfight behavior component, with methods to compute steering
        let playfight1 = agent1.addComponent(PsyanimPlayfightBehavior);
        playfight1.breakDuration = 2000;
        playfight1.fleeBehavior = flee1;
        playfight1.arriveBehavior = arrive1;
        playfight1.wanderBehavior = wander1;
        
        // the playfight agent is necessary to actually steer the agent using the 'behavior' component
        let playfightAgent1 = agent1.addComponent(PsyanimPlayfightAgent);
        playfightAgent1.playfightBehavior = playfight1;
        playfightAgent1.vehicle = vehicle1;

        /**
         *  Setup agent 2
         */

        // create entity with appropriate shapeParams
        let agent2 = this.addEntity('agent2', 600, 300, {
            shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE, 
            radius: 12, color: 0x0000ff
        });

        // the vehicle component is the fundamental movement component for steering behaviors
        let vehicle2 = agent2.addComponent(PsyanimVehicle);
    
        // the 'arrive' behavior is used for the 'charge' in playfight
        let arrive2 = agent2.addComponent(PsyanimArriveBehavior);
        arrive2.maxSpeed = 9;
        arrive2.maxAcceleration = 0.4;
        arrive2.innerDecelerationRadius = 12;
        arrive2.outerDecelerationRadius = 30;

        // flee allows us to keep the agents separated during wander
        let flee2 = agent2.addComponent(PsyanimAdvancedFleeBehavior);
        flee2.maxSpeed = 4;
        flee2.maxAcceleration = 0.2;
        flee2.panicDistance = 100;

        // wander uses 'seek' behavior to propel agent towards wander target
        let seek2 = agent2.addComponent(PsyanimSeekBehavior);
        seek2.maxSpeed = 4;
        seek2.maxAcceleration = 0.2;

        // add wander component, noting how it references the seek behavior
        let wander2 = agent2.addComponent(PsyanimWanderBehavior);
        wander2.seekBehavior = seek2;
        wander2.radius = 50;
        wander2.offset = 250;
        wander2.maxWanderAngleChangePerFrame = 20;

        // add playfight behavior component, with methods to compute steering
        let playfight2 = agent2.addComponent(PsyanimPlayfightBehavior);
        playfight2.breakDuration = 2000;
        playfight2.fleeBehavior = flee2;
        playfight2.arriveBehavior = arrive2;
        playfight2.wanderBehavior = wander2;
        
        // the playfight agent is necessary to actually steer the agent using the 'behavior' component
        let playfightAgent2 = agent2.addComponent(PsyanimPlayfightAgent);
        playfightAgent2.playfightBehavior = playfight2;
        playfightAgent2.vehicle = vehicle2;

        // setup targets for playfight agents
        playfightAgent1.setTarget(agent2);
        playfightAgent2.setTarget(agent1);
    }
```

Look at how much code it took to add two playfight agents to a scene - over 100 lines!

Moreover, there was a lot we had to know about the `Playfight algorithm`'s internal implementation details to configure the necessary components properly!

This added cognitive load isn't ideal when we're trying to focus our energy on designing the right experiments & parameter ranges for psychology & cognitive science research.

The good news is that we have a powerful tool to hide the complex internal implemention details of any behavior or components!

We call this tool `Entity Prefabs`.

## 3. `Entity Prefabs` to the rescue: Building a playfight scene the easy way

Let's create a second scene called `PlayfightSceneUsingPrefabs` using Psyanim-CLI: 

```bash
npx psyanim-cli -s PlayfightSceneUsingPrefabs
```

Open up `index.js`, register the new scene we just created and comment out the first scene (note that we can always uncomment / comment out scene registrations to quickly change which scene loads on app start):

```js
import { PsyanimApp } from 'psyanim2';

import PlayfightScene from './PlayfightScene';
import PlayfightSceneUsingPrefabs from './PlayfightSceneUsingPrefabs';

/**
 *  Setup Psyanim and PsyanimJsPsychPlugin
 */
// PsyanimApp.Instance.config.registerScene(PlayfightScene);
PsyanimApp.Instance.config.registerScene(PlayfightSceneUsingPrefabs);

PsyanimApp.Instance.run();
```

Now let's open up `PlayfightSceneUsingPrefabs.js` and update our `psyanim2` import statement to add the following classes as follows:

```js
import 
{ 
    PsyanimScene,
    PsyanimConstants,
    PsyanimPlayfightAgent,
    PsyanimPlayfightAgentPrefab

} from 'psyanim2';
```

Now it's time to setup the `playfight agents` in our `create()` method.

This time around, however, we aren't going to setup all the components for our agents directly.

Instead, we'll first create an `Entity Prefab` object which, as we mentioned before, is like a `blueprint` for easily creating agents with complex components & configurations.

For a `playfight agent`, the `prefab` we'll want to use is called the `PsyanimPlayfightAgentPrefab`.

Let's add the following code to the end of our `create()` method to create our prefab:

```js
    create() {

        super.create();

        // create agent prefab with initial parameter for the entity
        let agentPrefab = new PsyanimPlayfightAgentPrefab({
            shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE, 
            radius: 12, color: 0xff0000
        });
    }
```

// TODO: show how to use the API docs to see what properties the prefab exposes to the user