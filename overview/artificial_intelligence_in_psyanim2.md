# <ins>Artificial Intelligence in Psyanim-2</ins>

***Note: Full tutorial project + source code can be found [here](https://github.com/thefinnlab/hello-psyanim2) in the `artificial_intelligence` branch.***

## 1. Core concepts

`Psyanim-2` has a small, but continuously growing library of reusable `Artificial Intelligence` (AI) components that support agent `movement` and `decision-making` in real-time, interactive environments.

The `movement` and `decision-making` techniques & algorithms used in psyanim-2 have their origins in the application of `artificial intelligence` to the fields of video games and robotics.

The purpose of this tutorial is to familiarize the reader with some key terminology and concepts related to `AI Movement and Decision-Making` in `Psyanim-2` and provide resources for further research.

The types of `movement algorithms` used are primarily `kinematic movement algorithms` and `steering behaviors`.  Support has also been added for `pathfinding` with a `customizable navigation grid` and `static obstacles`.

`Psyanim-2` also has an `AI Decision-Making Framework` implemented using `Finite State Machines` (or FSMs), with the goal of making it easy to prototype & debug more complex `agent-based AI logic`.

## 2. Best References for AI Techniques used in Psyanim-2

***Before you proceed with the rest of the tutorial sections, if you don't have time to study the resources listed in this section in great detail, I would at least watch the following videos first to familiarize yourself with AI steering behaviors and finite state machines:***

- [Nature of Code: Autonomous Steering Agents Introduction](https://youtu.be/P_xJMH8VvAE?si=ichCvB3RTCJIHWI3)

- [Nature of Code: Seeking a Target](https://youtu.be/p1Ws1ZhG36g?si=k0oiIVLYjUoEARKL)

- [Finite State Machines](https://youtu.be/-ZP2Xm-mY4E?si=EqXvvlS4jHFMAzKb)

That said, if you do have more time and would like to do a deeper dive into relevant AI techniques, here are 3 resources that contain all you really need to know about creating AI agent behaviors in `psyanim2`:

***1. "AI for Games" by Ian Millington***

This book is the absolute best reference I've ever come across for all things Artificial Intelligence in games (or real-time, interactive simulations).

You can find latest edition of the book [here](https://www.amazon.com/AI-Games-Third-Ian-Millington/dp/1138483974).

It is a comprehensive overview of all forms of AI used in Psyanim-2, and much more.

***2. "The Beginner's Guide to Artificial Intelligence" by Penny de Byl***

The next best reference I've ever come across is this video course by Penny de Byl.  

You can find the course [here](https://www.udemy.com/course/artificial-intelligence-in-unity/).

***I actually recommend this course as the first place to start, even before Millington's book, as she does such a great job of breaking everything down into very concise, yet effective visual explanations of each topic.***

Even though de Byl's course examples are designed for C# in the Unity engine, it's not hard to follow along and translate the algorithms to javascript in Psyanim-2, since Psyanim-2 also has a component architecture and was largely inspired by Unity's monobehavior component design.

The course is a comprehensive overview of all forms of AI used in Psyanim-2, and much more.

***3. Daniel Schiffman's "The Nature of Code" book and video lectures***

This book and video lecture series is one of the best introductions to AI steering behaviors I've seen, and though it's built on a different real-time graphics framework, the algorithms are implemented in javascript in a very pseudo-code-esque fashion that makes it easy to translate to Psyanim-2 or Phaser.

Here's a link to the [book](https://natureofcode.com/autonomous-agents/).

Here's a link to the [video lectures](https://youtu.be/P_xJMH8VvAE?si=XOgqMA1L8bJlxea2).

## 3. AI Steering Architecture in Psyanim 2

***NOTE: this tutorial assumes some familiarity with the `Seek` steering behavior, as the `Arrive` behavior is just a variant of that.  For more information, see the previous section.***

In this section, we'll get some hands-on experience with the `steering architecture` in `Psyanim 2` by designing an with an `AI agent`.

The `AI agent` we'll create will `patrol` back and forth between a few points in the world.

We will give the AI agent a 'brain' later to decide how to switch between different behaviors (known as `decision-making` in game AI) depending on what's happening in the environment, but for this section, let's just get the AI agent patrolling between a few points using a `pathfollowing` steering behavior, so we can learn a bit about the `steering architecture` in general.

Let's get started by creating a new, empty `psyanim2` project using `psyanim-cli` (make sure `NodeJS v18+`, `Git`, and `psyanim-cli` are installed first):

```bash
psyanim init
```

Let's create an new scene with `psyanim-cli`:

```bash
psyanim asset:scene HelloAIScene -o ./src/scenes
```

You can delete the `EmptyScene.js`, remove all the commented out code from the `index.js` and add a trial for `HelloAIScene.js` to it.

---

Open `HelloAIScene.js` and let's start building out our scene.  We can delete the `navgrid` property and set the `wrapScreenBoundary` property to `true`.

***IMPORTANT NOTE: make sure you set the `wrapScreenBoundary` property to `true`, or our agent's AI behaviors won't work as expected b.c. the world boundaries can't be avoided.***

Update your `psyanim2` package imports at the top so they include the following classes:

```js
import { 
    
    PsyanimConstants,

    PsyanimVehicle,
    PsyanimArriveBehavior,
    PsyanimArriveAgent,
    PsyanimPathFollowingAgent,

} from 'psyanim2';
```

Now, let's add a single triangle-shaped entity as our `agent` in the scene:

```js
...
{
    name: 'agent',
    initialPosition: { x: 100, y: 200 },
    shapeParams: {
        shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE,
        color: 0x0000ff,
        base: 12, altitude: 20
    },
    components: []
},
...
```

Now, we need to add some `components` to this `agent` to get it `patrolling` using a `pathfollowing behavior`.

First, let's add the `PsyanimVehicle` component:

```js
...
components: [
  { type: PsyanimVehicle },
]
...
```

The `PsyanimVehicle` component provides the foundational physics-based interface for applying steering forces to any entity.  All steering behaviors require this component.

Next, we'll want to add a `PsyanimArriveBehavior` component:

```js
...
{
    type: PsyanimArriveBehavior,
    params: {
        maxSpeed: 6,
        innerDecelerationRadius: 4,
        outerDecelerationRadius: 50
    }
},
...
```

Don't worry too much about the parameters for the `PsyanimArriveBehavior` here, as those were tuned just to make the motion for this example look nice, but aren't critical to a general understanding of the `steering architecture`.

The `PsyanimArriveBehavior` component is responsible for *calculating* `steering forces` for an `entity`.

The `PsyanimArriveBehavior` is *not* responsible for *applying* these `steering forces` each frame.  

This separation of concerns is critical to note because these basic `steering behaviors` are designed to be used as building blocks to form more complex behaviors, which will switch between different simpler steering behaviors at runtime.

Thus, to make these `behaviors` reusable across many other more complex behaviors, the `behavior components` themselves do *not* apply `steering forces` to the `PsyanimVehicle`.

They simply compute a `steering force` whenever requested by *something else* and that *something else* is responsible for choosing a force to apply to the `PsyanimVehicle`.

What is this *something else* that requests `steering forces` from `behavior components` each frame and then decides which one(s) to apply to the `PsyanimVehicle`?

The convention in this `steering architecture` is to create an `*Agent` component to do this, with the `*` part of the name denoting a prefix that is specific to the behavior of the `agent`.

For our purposes here, we will use the `PsyanimArriveAgent` component to apply `steering forces` computed by the `PsyanimArriveBehavior` component to the `PsyanimVehicle` each simulation frame:

```js
...
{
    type: PsyanimArriveAgent,
    params: {
        arriveBehavior: {
            entityName: 'agent',
            componentType: PsyanimArriveBehavior
        },
        vehicle: {
            entityName: 'agent',
            componentType: PsyanimVehicle
        }
    }
},
...
```

Notice that, in the `component definition` above, the `PsyanimArriveAgent` component references the `PsyanimArriveBehavior` and `PsyanimVehicle` components we attached to the same `agent` entity.

This is because, each simulation frame, the `PsyanimArriveAgent` will query the `PsyanimArriveBehavior` for the appropriate `steering force` and then apply it to the `PsyanimVehicle`.

Sometimes, it is helpful to create `*Agent` components which are composed of other `*Agent` components, just to avoid code duplication.

For instance, to make our `agent entity` follow a path consisting of several points in the world, we will need to add a `PsyanimPathFollowingAgent` component which is composed of (or depends on) the `PsyanimArriveAgent` component.

The `PsyanimPathFollowingAgent`'s algorithm updates the `target` of the `PsyanimArriveAgent` each frame so that the `agent entity` is moving in the direction of the next desired point in the world.

Let's add the `PsyanimPathFollowingAgent` to our `agent` entity's `components array` as follows:

```js
...
{
    type: PsyanimPathFollowingAgent,
    params: {
        currentPathVertices: [
            { x: 100, y: 200 },
            { x: 400, y: 50 },
            { x: 700, y: 200 },
        ],
        arriveAgent: {
            entityName: 'agent',
            componentType: PsyanimArriveAgent
        },
        targetPositionOffset: 50
    }
},
...
```

Notice that, as we discussed, the `PsyanimPathFollowingAgent` has a reference to the `PsyanimArriveAgent` component on the same entity.  This is because the `PsyanimPathfollowingAgent` works by controlling the `target` of the `PsyanimArriveAgent` component.

This is a common approach to building more complex behaviors - by combining simpler behaviors within a `state machine` (state machines are discussed more in the next section).

Also note the `path` the agent will follow is defined simply as a `list of points`.  The agent will follow this `path` from start point to end point, and then (by default) reverse direction every time it reaches the end of a path and continue `patrolling` it.

Great work!  By this point, you should have a blue, triangle-shaped agent patrolling back and forth along the path defined by the 3 points we added to our `PsyanimPathFollowingAgent` component.

---

In summary, all `agents` with `steering behaviors` use the `PsyanimVehicle` component to apply `steering forces` to an `entity`.

The `*Behavior` components do *not* apply steering forces to the `PsyanimVehicle`, but they provide an interface to request a `steering force` computation, returning the steering force to the caller.

The `*Agent` components are responsible for querying the `*Behavior` components for steering forces and then deciding how those steering forces get applied to the `PsyanimVehicle` each frame.

This separation of concerns allows for `*Behavior` components to be *reused* in other `*Behaviors` or `*Agents`.

## 4. Finite State Machines in Psyanim 2

In order to give the `agent` entity the ability to switch between behaviors, or `make decisions`, based on the state of the world, we need some sort of a structure or pattern to define the logic for these decisions.

Prior the advent of [behavior trees](https://en.wikipedia.org/wiki/Behavior_tree_(artificial_intelligence,_robotics_and_control)), the structure commonly used in video games for `agent decision-making` was `finite state machines`.

While `behavior trees` have certainly become the de facto standard for most games today due to their flexibility and ease of reuse, they are often still combined with `finite state machines` to produce even more sophisticated behaviors.

`Psyanim 2`'s decision-making framework offers `finite state machines` (or *FSMs*) via the `PsyanimFSM` and `PsyanimFSMState` classes.

One big advantage of a state machine is that it can be easily visualized and reasoned about using simple boxes and arrows.

The fact that state machines lend themselves so easily to clear visual representations means that subject-matter experts can participate in AI design, regardless of their level of comfort with programming.

---

When designing a state machine for any purpose, I always start with a pencil and a piece of paper, sketching out the `states` represented by `boxes` and `allowed transitions` represented by `arrows`.

I highly recommend that, before trying to code up any state machine, you start by sketching out a simple diagram of it's design to make it easy to reason about as you implement it.

<p align="center" style="font-size: 12px;">
    <img src="./imgs/item_patrol_fsm.jpg"/>
    <!-- <br>Your caption goes here -->
</p>

Looking at the `state diagram` above, you can see that this `finite state machine`, named `MyItemPatrolFSM`, consists of `2 states`: 

- MyPatrolState
- MyMoveToItemState

The primary task of this state machine is to have an agent patrol along a fixed path until an item appears in the scene, at which point it will move to the item to collect it, after which it will return to it's patrol until another item appears.

In the `MyPatrolState`, the agent will just patrol between a set of pre-defined points (already setup in the previous tutorial section).

In the `MyMoveToItemState`, the agent will move to an item's location in the scene to collect it when it appears.

The `entry point` to the state machine, or `initial state`, is set to be the `MyPatrolState`.

Between the `states` on the diagram, we have `arrows` indicating the `allowed transitions` from each state to other states.

The `circled numbers` next to each `arrow` on the diagram represent `transition conditions`, each of which are detailed below the diagram.

For now, let's just discuss the general flow of state changes in this state machine with a few bullet points here:

- Agent starts out in the initial state of `MyPatrolState`, where the agent will be patrolling along a set of pre-defined points in the world.

- When an item appears in the scene, the agent transitions to the `MyMoveToItemState` to collect the item.

- Once the item is collected, the agent transitions back to the `MyPatrolState` until another item appears in the scene.

As with all state machines in general, in this `MyItemPatrolFSM` state machine, the agent is always in one, and only one, of these 2 possible states.

---

To make this work in practice, we'll need to create these two states, define the agent behaviors in each one, as well as defining the conditions for allowed transitions between them.

Luckily, `psyanim-cli` can be used to setup all the boiler plate we need to build our `state machine`.

---

Run the following command in terminal to create the 2 `states` of our `state machine`:

```bash
psyanim asset:fsmstate MyPatrolState MyMoveToItemState -o ./src/basic_hfsm/item_patrol_fsm
```

You should see 2 source files show up under the `/src/basic_hfsm/item_patrol_fsm` directory - one for each state.

If you open up `MyPatrolState.js`, you'll see that the state is a `javascript class` with a `contructor` and `6 methods`: `afterCreate`, `enter`, `exit`, `run`, `onPause`, `onStop`, and `onResume`.

We can ignore the `onPause`, `onStop`, and `onResume` methods for now, as they will be discussed in the next section.

The `constructor` is only executed once when the state is first created to be added to the state machine.

The `enter()` method is executed once every time the entity `enters` that state.

The `exit()` method is executed once every time the entity `exits` that state.

The `run(t, dt)` method is executed once every simulation frame, so long as the state is `active` (meaning the entity is currently in that state).

All `state transitions` should be added in the `constructor` of the state class, since we only want to add these transitions *once* for each state machine.

Every state machine maintains `state variables` that can be read / written to from any `state` via a set of APIs in the `PsyanimFSMState`.  These `state variables` can be used to trigger transitions.

The general workflow for implementing a `PsyanimFSMState` is as follows:

- Create the state using `psyanim-cli`
- Add transition conditions based on `state-variable` values in the `constructor` of the state class
- In `enter`, `exit`, and `run` methods, do any work and update `state variables` of the `state machine` as necessary, triggering state transitions when appropriate.

---

Go ahead and copy the raw contents of the finished source files for each state into your state files.  The links are:

- [MyPatrolState.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/basic_hfsm/item_patrol_fsm/MyPatrolState.js)
- [MyMoveToItemState.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/basic_hfsm/item_patrol_fsm/MyMoveToItemState.js)

Let's look at `MyPatrolState.js` in greater detail.  At the end of the `constructor` you can see the transition to the `MyMoveToItemState` is added.

Each call to `addTransition()` accepts 3 arguments:

- The `type` of the state to transition to.
- The name of the `state variable` the transition is based on
- The `condition` on which the value of the variable will `trigger` a `transition`

Next, take a look at the `enter()` and `run()` methods of our `MyPatrolState` class.

Notice the `enter()` method just sets a `state variable`, enables & configures the `arrive agent` component, and then sets the entity's color to a value specific to this state (for visualization purposes).

In the `run()` method, we query the current `PsyanimScene` object to see if there is an `item` in the scene.  If there is, we set the `itemInScene` state variable to `true`.

The `enter()` and `run()` methods of any state do not need to explicitly check `state variables` to `trigger` state transitions.  This happens automatically in the `PsyanimFSM` state machine.

If you have time, as an exercise, check out both states' source code and try to understand how the code there relates to what we see in our state diagram sketch from earlier.

---

The last thing we need to do is just create the `PsyanimFSM` component on the `agent` entity in our scene and add our states to it!

To do this, we'll create a `MyItemPatrolFSM` component that encapsulates the state machine creation.  Back in the terminal, run:

```bash
psyanim asset:fsm MyItemPatrolFSM -o ./src/basic_hfsm/item_patrol_fsm
```

Let's open up the `MyItemPatrolFSM.js` and take a peek at's structure.

Notice that `MyItemPatrolFSM` inherits from `PsyanimFSM`, which is itself a `PsyanimComponent`.

All `PsyanimFSM` classes inherit a set of methods which can be overriden for different purposes, including: `afterCreate`, `onPause`, `onStop`, `onResume`, and `update`.

The `afterCreate` and `update` methods are just overrides for the base `PsyanimComponent` methods.

We will discuss `onPause`, `onStop` and `onResume` in the next section.  We can actually remove them from this FSM since they won't be needed here.

Go ahead and copy the finished source file into your `MyItemPatrolFSM.js`:

- [MyItemPatrolFSM.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/basic_hfsm/item_patrol_fsm/MyItemPatrolFSM.js)

Notice the only two methods this `MyItemPatrolFSM` class implements are the `constructor` and `update`.

In the `constructor`, we add our two states to the `MyItemPatrolFSM` and then set the `initial state`, which is the first state that will be active when the FSM starts up.

In the `update` method of `MyItemPatrolFSM`, a `distanceToTarget` state variable is computed and set every frame.

You can safely ignore this for now, as it will come in handy later. It is good to make note of how `state variables` can be set for an FSM, though.

---

Let's add this `MyItemPatrolFSM` component to our `HelloAIScene` definition by first importing it at the top of the file and adding this `component definition` to the `agent` entity:

```js
{
    type: MyItemPatrolFSM,
}
```

We'll also need to add a `PsyanimSensor` component to this `agent` entity, as the `MyMoveToItemState` expects the entity to have one attached.

To do so, update the `psyanim2` import statement at the top of the `HelloAIScene` definition to also import `PsyanimSensor`.

Then, add the following `component definition` to the `agent` entity:

```js
{
    type: PsyanimSensor,
    params: {
        bodyShapeParams: {
            shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
            radius: 24                
        }
    }
}
```

--- 

So, we now have our FSM setup, but if you reload your experiment in the browser, the agent never leaves the `MyPatrolState`.

This is because the `transition condition` for leaving that state is triggered by the condition that an `item` exists in the scene.

To get an `item` to be added to the scene periodically, we'll create a `MyItemSpawner` component with `psyanim-cli`:

```bash
psyanim asset:component MyItemSpawner -o ./src/components
```

Open up `MyItemSpawner.js` and replace it's contents with the finished source code here:

- [MyItemSpawner.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/components/MyItemSpawner.js)

All this component does is add an `item` to our `scene` at a particular location with a particular frequency.

Let's add this component to a new entity in our `HelloAIScene` definition's `entities array` using the following `entity definition` (be sure to import `MyItemSpawner.js` at the top of the file):

```js
{
    name: 'itemSpawner',
    components: [
        {
            type: MyItemSpawner
        }
    ]
},
```

Reload your `experiment` in the `browser` and you should see the `agent` entity `patrolling` until an `item` is spawned in the scene, at which point it will `move to` that item to collect it before returning to it's original patrol.

Great work - you've created your first `finite-state machine` in `psyanim-2`!

## 5. Interactive FSM Agent

Now let's create a `finite-state machine` for an `agent` that responds to a `player-controlled entity`, according to the following `state diagram`:

<p align="center" style="font-size: 12px;">
    <img src="./imgs/flee_fsm.jpg"/>
    <!-- <br>Your caption goes here -->
</p>

In the `MyFleeState`, the agent will `flee` from a `target`.  In this case, the target will be a `player-controlled entity`.

In the `MyIdleState`, the agent will not execute any `steering behaviors`, but may slowly glide to a stop and wait there for as long as the state is active.

---

First, let's add a `player-controlled entity` to our `scene definition`, and add a few extra components to our `agent` entity to allow it to execute a `flee behavior` too.

Update your `psyanim2` imports at the top of `HelloAIScene.js` to include `PsyanimPlayerController`.  While you're at it, go ahead and add `PsyanimFleeBehavior` and `PsyanimFleeAgent` to your `psyanim2` imports too.

Then, adding the `player-controlled entity` to our `entities` array in the `scene definition` is as simple as:

```js
...
{
    name: 'player',
    initialPosition: { x: 400, y: 550 },
    shapeParams: {
        shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE,
        color: 0xff0000,
        base: 12, altitude: 20
    },
    components: [
        {
            type: PsyanimPlayerController
        }
    ]
},
...
```

Now, we need to add a `PsyanimFleeBehavior` and `PsyanimFleeAgent` component to our `agent` entity so it is able to execute a `flee behavior` when needed:

```js
...
{
    type: PsyanimFleeBehavior,
    params: {
        maxSpeed: 8,
        maxAcceleration: 0.3,
        panicDistance: 150
    }
},
{
    type: PsyanimFleeAgent,
    params: {
        fleeBehavior: {
            entityName: 'agent',
            componentType: PsyanimFleeBehavior
        },
        vehicle: {
            entityName: 'agent',
            componentType: PsyanimVehicle
        },
        target: {
            entityName: 'player'
        }
    }
},
...
```

Next, let's setup our `finite-state machine` to actually have the agent move between `fleeing` and `idling`.

Back in a terminal, from the project root directory, run the following command to create our 2 state classes:

```bash
psyanim asset:fsmstate MyFleeState MyIdleState -o ./src/basic_hfsm/flee_fsm
```

You should see `MyFleeState.js` and `MyIdleState.js` created under `./src/basic_hfsm/flee_fsm`.

Replace the contents of these new files with the finished source code here:

- [MyFleeState.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/basic_hfsm/flee_fsm/MyFleeState.js)
- [MyIdleState.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/basic_hfsm/flee_fsm/MyIdleState.js)

Now, let's create the actual FSM class, `MyFleeFSM`, which will control how the agent transitions in and out of these states:

```bash
psyanim asset:fsm MyFleeFSM -o ./src/basic_hfsm/flee_fsm
```

Replace the contents of the `MyFleeFSM.js` file with the completed source code:

- [MyFleeFSM.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/basic_hfsm/flee_fsm/MyFleeFSM.js)

There really aren't any fundamentally new concepts in these 3 files here, with only 2 files for state definitions and 1 for the FSM definition which ties together the states.

If you're interested in exploring the internal workings of these FSMs, it's left here as an exercise to use the concepts we've learned to study the code and see what it's doing.

At a high-level, the `MyFleeFSM` class just adds the `MyIdleState` and `MyFleeState` to a state machine, with the `MyFleeState` as the initial state and a tad bit of configuration.

In `MyIdleState`, the `agent` has no steering forces applied to it at all.

In `MyFleeState`, the `agent` flees until it is outside of the (`paniceDistance` + `safteyDistance`).

To wrap up the `MyFleeFSM`, let's update our imports to include `MyFleeFSM` and add it to our `agent` entity's `components array` as follows:

```js
{
    type: MyFleeFSM
},
```

Great work - you've just finished creating your `MyFleeFSM`!  

However, this isn't quite ready to work since we can't simply have 2 `PsyamimFSMs` on the same `agent` without a `PsyanimBasicHFSM` to tie them together.

## 6. Hierarchical FSMs in Psyanim-2

In this section, we'll learn about `hierarchical FSMs (HFSMs)` by building one that combines the two `finite state machines` we created in the previous two sections.

While `psyanim2` does not have full support for `hierarchical FSMs` at the moment, it does support a simpler version as described by Ian Millington in his book `Artificial Intelligence for Games`:

> An implementation of hierarchical state machines could be made significantly simpler than this by requiring that transitions can only occur between states at the same level. With this limitation in force, all the recursion code can be eliminated.
> If you don’t need crosshierarchy transitions, then the simpler version will be easier to implement. 

This simpler hierarchical FSM, called `PsyanimBasicHFSM`, is implemented similar to [pushdown automata](https://en.wikipedia.org/wiki/Pushdown_automaton), but with entire sub-state machines rather than just states, so a 'history' of states is automatically kept internally in the underlying stack data structure and the hierarchy is enforced by position within the stack.

That said, `psyanim2` can always be extended with full-featured `heirarchical FSMs` or `behavior trees` (or a hybrid of the two) at a later time should the need arise.

---

Now, we have two state machines, `MyItemPatrolFSM` and `MyFleeFSM`, attached to our `agent` entity as `components`.

Next, we will create a `basic hierarchical state machine` which will run both our our `PsyanimFSMs` as `sub-state machines`.

A `PsyanimBasicHFSM` is capable of running any one of multiple configured `PsyanimFSMs` at a given time, and switching between each one at appropriate times.

An `agent` with a `PsyanimBasicHFSM` component attached is required to always be running exactly 1 `PsyanimFSM` at any given time as a `sub-state machine`.

Every `PsyanimBasicHFSM` has an `initialSubStateMachine` property which defines the `sub-state machine` it will initially execute.

An `interrupt` is a mechanism for switching between `sub-state machines` in a `PsyanimBasicHFSM`, and it is `triggered` by a user-defined condition.

A `PsyanimBasicHFSM` can have many `interrupt conditions` defined which trigger a switch from one `sub-state machine` to another.

The `PsyanimBasicHFSM` maintains a stack data structure internally, indicating the priority of `sub-state machines` to execute.

When an `interrupt` is `triggered`, the currently executing `sub-state machine` is either `stopped` or `paused` and the target `sub-state machine` is pushed onto the stack and `resumed`.

Any time a `sub-state machine` is `stopped`, it is removed from the stack.

Any time a `sub-state machine` is `paused`, it remains on the stack.

Any time an `interrupt` is triggered, the `target sub-state machine` is `resumed` and pushed onto the stack.

A `sub-state machine` that is `resumed` from a `paused` state will continue from where it previously left off, with all of it's `state variables` in tact.

A `sub-state machine` that is `resumed` from a `stopped` state will restart it's execution from it's `initial state`, with all `state variables` reset also.

---

As with any `finite state machine`, we'll begin the design of our `hierarchical state machine`, `MyBasicHFSM`, with a `state diagram` sketch:

<p align="center" style="font-size: 12px;">
    <img src="./imgs/basic_hfsm.jpg"/>
    <!-- <br>Your caption goes here -->
</p>

**NOTE: this diagram does not denote when a sub-state machine should be `paused` vs. `stopped`. A visual convention for this is in the works - stay tuned!**

Here, we can see the two `sub-state machines` of `MyBasicHFSM` are: `MyItemPatrolFSM` and `MyFleeFSM`.

Each of these `sub-state machines` behave just the same as they would have independently.

The purpose of `MyBasicHFSM` is to tie these two `sub-state machines` into a larger `state machine` with rules for switching between them, defined as `interrupts` or `interrupt conditions`.

Similar to how `arrows` were used on our previous `state diagrams` to denote allowed `transitions` between states, `arrows` are also used to denote `interrupts` which cause a `hierarchical state machine` to switch from one `sub-state machine` to another.

The `interrupts`, however, are identified *by convention* in `psyanim-2 state diagrams` by a prefix of capital `I`.

In our `MyBasicHFSM` machine, we can see there are two `interrupts`, labeled `I1` and `I2`, with their `interrupt conditions` defined at the bottom of the diagram.

The `initialSubStateMachine` of this `MyBasicHFSM` is the `MyItemPatrolFSM`.  This is the sub-state machine that will be active initially, and the first state-machine on the HFSM's internal stack.

In accordance with the `interrupt conditions` for `MyBasicHFSM`, if an attacker is within `panicDistance` of this agent, `I1` is triggered and `MyBasicHFSM` will `pause` the `MyItemPatrolFSM` and `resume` the `MyFleeFSM`, pushing it onto the top of the internal sub-state machine `stack`.

When the `MyFleeFSM` is running, if the `agent` has been in it's `idle state` for a duration of `returnToPatrolTime`, `I2` is triggered and `MyBasicHFSM` will `stop` the `MyFleeFSM`, popping it off the stack, and then `resume` the `MyItemPatrolFSM`.

It is up to the `AI designers / developers` to design the `interrupt conditions` in such a way that the `hierarchical state machine` will only switch between states as desired.

---

Now that we've got a better understanding of the high-level operation of the `hierarchical FSM`, let's go ahead and implement it in our project.  Run the following command from the project root dir:

```bash
psyanim asset:component MyBasicHFSM -o ./src/basic_hfsm
```

Open up `MyBasicHFSM.js` and replace it's contents with the following finished source code:

- [MyBasicHFSM.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/basic_hfsm/MyBasicHFSM.js)

We updated the code so that `MyBasicHFSM` inherits from `PsyanimBasicHFSM`, which is also a `PsyanimComponent`.

In the `afterCreate()` method, we add and configure the `sub-state machines` of this `PsyanimBasicHFSM`, which are `MyItemPatrolFSM` and `MyFleeFSM`.

We then define our two interrupts using the `this.addInterrupt(...)` method of `PsyanimBasicHFSM`.

Each `addInterrupt` call takes a minimum of 3 parameters, with an optional fourth:

- `sourceFSMType`: the type of the FSM from which the interrupt can be triggered

- `variableKey`: the key of the state variable against which the `interrupt condition` will be checked

- `interruptCondition`: the condition on which the interrupt will occur. A function with 1 arg - the `state variable value`

- `destinationFSMType`: the sub-state machine type that this HFSM will push onto the stack if interrupt is triggered.  If not provided, this interrupt will remove the current FSM from the stack and run the one below it.

Finally, the `MyBasicHFSM` component must define the `initialSubStateMachine` which will be the first sub-state machine pushed onto the stack and the initial sub-state machine that executes on scene start.

---

Back in our `HelloAIScene` scene definition, let's add the following `component definition` to our `agent` entity's components array:

```js
{
    type: MyBasicHFSM,
    params: {
        target: {
            entityName: 'player',
        },
        
        fleePanicDistance: 150,
        fleeSafetyDistance: 50,

        returnToPatrolTime: 1500
    }
}
```

The `fleePanicDistance` and `fleeSafetyDistance` are used to determine at what distance the `agent` should start/stop fleeing from the `player`.

The `returnToPatrolTime` is the amount of time the `agent` needs to remain in it's `idle` state without fleeing before it can return to it's `MyItemPatrolFSM` task of patrolling & collecting items.

Great work! If you reload the experiment in your browser, you should see two entities.

One is the `player` entity and can be moved around with the `W`, `A`, `S`, and `D` keys on your keyboard.  The `player` entity is *always* a red and never changes color.

The other entity is our `agent` and should be patrolling back and forth unless the player moves close to it.  It will change colors depending on it's state, as we've seen previously in our code.

When the `player` approaches the `agent`, it's `MyItemPatrolFSM` will be interrupted and it's `MyFleeFSM` will be pushed onto the HFSM stack to be executed.

When an `item` appears in the scene (as a green circle near the center of the canvas), as long as the `agent` is in the `MyItemPatrolFSM` sub-state machine, it will switch to the `MyMoveToItemState` and collect the item before returning to a `patrol`.

---

As an exercise, to test our intuition against what we observe in practice, let's remove the interrupt that takes the `MyBasicHFSM` from the `MyItemPatrolFSM` sub-state machine to the `MyFleeFSM` sub-state machine and observe the resulting behavior.

We should expect to see the `agent` never switching to the `MyFleeFSM`, regardless of how close the `player` entity gets to it.

Let's update our `MyBasicHFSM::afterCreate()` method to test this case:

```js
    afterCreate() {

        super.afterCreate();

        // configure and add sub-state machines
        this._itemPatrolFSM = this.entity.getComponent(MyItemPatrolFSM);
        this._itemPatrolFSM.target = this.target;

        this._fleeFSM = this.entity.getComponent(MyFleeFSM);
        this._fleeFSM.target = this.target;
        this._fleeFSM.fleePanicDistance = this.fleePanicDistance;
        this._fleeFSM.fleeSafetyDistance = this.fleeSafetyDistance;

        this.addSubStateMachine(this._itemPatrolFSM);
        this.addSubStateMachine(this._fleeFSM);

        // add interrupts
        // this.addInterrupt(MyItemPatrolFSM, 'distanceToTarget', (value) => value < this.fleePanicDistance, MyFleeFSM);
        // this.addInterrupt(MyFleeFSM, 'idleTime', (value) => value > this.returnToPatrolTime);

        // setup initial substate machine to run
        this.initialSubStateMachine = this._itemPatrolFSM;
    }
```

If you reload the experiment in your browser, you should see that the `agent` does indeed remain in the `MyPatrolFSM` at all times, irrespective of it's proximity to the `player`.

---

As a final exercise, to test our intuitions even further, let's keep the `interrupts` commented out in the `MyBasicHFSM::afterCreate()` method and set the `initialSubStateMachine` to the `MyFleeFSM` component, as follows:

```js
    afterCreate() {

        super.afterCreate();

        // configure and add sub-state machines
        this._itemPatrolFSM = this.entity.getComponent(MyItemPatrolFSM);
        this._itemPatrolFSM.target = this.target;

        this._fleeFSM = this.entity.getComponent(MyFleeFSM);
        this._fleeFSM.target = this.target;
        this._fleeFSM.fleePanicDistance = this.fleePanicDistance;
        this._fleeFSM.fleeSafetyDistance = this.fleeSafetyDistance;

        this.addSubStateMachine(this._itemPatrolFSM);
        this.addSubStateMachine(this._fleeFSM);

        // add interrupts
        // this.addInterrupt(MyItemPatrolFSM, 'distanceToTarget', (value) => value < this.fleePanicDistance, MyFleeFSM);
        // this.addInterrupt(MyFleeFSM, 'idleTime', (value) => value > this.returnToPatrolTime);

        // setup initial substate machine to run
        this.initialSubStateMachine = this._fleeFSM;
    }
```

If you now reload the experiment in your browser again, you'll see that the agent starts out in the `MyFleeFSM` and never gets interrupted to return to the `MyItemPatrolFSM`.

Furthermore, the agent starts out in the `MyIdleState` of the `MyFleeFSM`, and does transition to `MyFleeState` when the `player` is close enough nearby.

---

In the previous two exercises involving `MyBasicHFSM`, where we removed `interrupts` and changed the `initialSubStateMachine`, we observed an important property of the `PsyanimBasicHFSM`, which is it's modularity.

Not only is this easier to understand, but we are able to add/remove interrupts and sub-state machines with relative ease, once they are designed and implemented.

This modularity makes for a more flexible workflow that enables rapid prototyping and building highly configurable, complex agent behaviors.

---

In this tutorial, we've learned about the `AI Steering Architecture` and `Decision-Making Framework` in `psyanim-2` by creating an interactive experiment that uses `steering behaviors` and `finite-state machines` to control an `AI Agent`.

To continue this journey into the world of `game AI`, check out the resources in [section 2](/overview/artificial_intelligence_in_psyanim2.md#_2-best-references-for-ai-techniques-used-in-psyanim-2) of this tutorial!