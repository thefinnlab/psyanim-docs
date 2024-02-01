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

Here's a link to the [book](https://natureofcode.com/book/chapter-6-autonomous-agents/).

Here's a link to the [video lectures](https://youtu.be/P_xJMH8VvAE?si=XOgqMA1L8bJlxea2).

## 3. AI Steering Architecture in Psyanim 2

***NOTE: this tutorial assumes some familiarity with the `Seek` steering behavior, as the `Arrive` behavior is just a variant of that.  For more information, see the previous section.***

In this section, we'll get some hands-on experience with the `steering architecture` in `Psyanim 2` by designing an `interactive scene` with an `AI agent` and a `player-controlled agent`.

The `AI agent` will `patrol` back and forth between a few points in the world, and when approached by the `player-controlled agent`, the `AI agent` will `flee` from it, waiting for some time at a certain distance away before attempting to return back to it's `patrol`.

We will give the AI agent a 'brain' later to decide how to switch between behaviors (known as `decision-making` in game AI), but for this section, let's just get the AI agent patrolling between a few points using a `pathfollowing` steering behavior, so we can learn a bit about the `steering architecture` in general.

Let's get started by creating a new, empty `psyanim2` project using `psyanim-cli` as we did in the [Hello Psyanim 2.0 tutorial](/overview/hello_psyanim_2.md#2-creating-a-new-psyanim-2-project).

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
    PsyanimPlayerController,

    PsyanimVehicle,
    PsyanimArriveBehavior,
    PsyanimArriveAgent,
    PsyanimPathFollowingAgent,

    PsyanimFleeBehavior,
    PsyanimFleeAgent

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

While `PsyanimFSM` does not support `hierarchical FSMs` at the moment, the framework can always be extended with `heirarchical FSM`s or `behavior trees` or a hybrid of the two at a later time.

---

One big advantage of a state machine is that it can be easily visualized and reasoned about using simple boxes and arrows.

When designing a state machine for any purpose, I always start with a pencil and a piece of paper, sketching out the `states` represented by `boxes` and `allowed transitions` represented by `arrows`.

I highly recommend that, before trying to code up any state machine, you start by sketching out a simple diagram of it's design to make it easy to reason about as you implement it.

<p align="center" style="font-size: 12px;">
    <img src="./imgs/ai_fsm_design.jpg"/>
    <!-- <br>Your caption goes here -->
</p>

Looking at the state machine diagram above, you can see that this `finite state machine`, named `MyPatrolFleeAgentFSM`, consists of `3 states`: 

- MyPatrolState
- MyFleeState
- MyIdleState

In the `MyPatrolState`, the agent will just patrol between a set of pre-defined points (already setup in the previous tutorial section).

In the `MyFleeState`, the agent will `flee` from a `target`.  In this case, the target will be a `player-controlled entity`.

In the `MyIdleState`, the agent will not execute any `steering behaviors`, but may slowly glide to a stop and wait there for as long as the state is active.

The `entry point` to the state machine, or `initial state`, is set to be the `MyPatrolState`, as indicated by the `PsyanimScene::create()` method pointing to it.

Between the `states` on the diagram, we have `arrows` indicating the `allowed transitions` from each state to other states.

We could've drawn the diagram a bit larger so as to include a small `transition condition` to be written next to each arrow.

For now, let's just discuss the general flow of state changes in this state machine with a few bullet points here:

- Agent starts out in the initial state of `MyPatrolState`, where the agent will be patrolling along a set of pre-defined points in the world.

- When the `player-controlled entity` gets within a certain distance of the agent, it will transition to the `MyFleeState` where it will quickly accelerate away from the `player-controlled entity`.

- When the `player-controlled-entity` is far enough away from the `agent` entity, the `agent` will transition to the `MyIdleState`.

- In the `MyIdleState`, the `agent` waits a period of time before attempting to transition back to the `MyPatrolState`.  If, while waiting, the `player-controlled entity` gets too close to the `agent` entity, the `agent` will transition back to the `MyFleeState`.

In this `MyPatrolFleeAgentFSM` state machine, the agent is always in one, and only one, of these 3 states.

---

To make this work in practice, we'll need to create these three states, define the agent behaviors in each one, as well as defining the conditions for allowed transitions between them.

Luckily, `psyanim-cli` can be used to setup all the boiler plate we need to build our `state machine`.

First, however, let's add a `player-controlled entity` to our `scene definition`, and add a few extra components to our `agent` entity to allow it to execute a `flee behavior` too.

Adding the `player-controlled entity` to our `entities` array in the `scene definition` is as simple as:

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

Now, we just need to add a `PsyanimFleeBehavior` and `PsyanimFleeAgent` component to our `agent` entity so it is able to execute a `flee behavior` when needed:

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
    enabled: false,
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

Notice that, in the above snippet, the `PsyanimFleeAgent` component has it's `enabled` property set to `false`.  This is because it would conflict with the `PsyanimPathFollowingAgent` if we had them both enabled simultaneously.

So, we disable the `PsyanimFleeAgent` to start out, and our `PsyanimFSM`, will be responsible for switching between the `PsyanimFleeAgent` and the `PsyanimPathfollowingAgent` behaviors as appropriate later at runtime.

By this point, if you reload the experiment in your browser, you should see two agents: a `red` player-controlled agent and `blue` AI-controlled agent.

You should be able to move the `red` player-controlled agent around in the scene using the `W`, `S`, `A`, and `D` keys.

However, if you move the `player-controlled agent` into the `AI-controlled agent`, the AI controlled agent doesn't flee and the two entities just collide.

This is not what we want.  So, let's build out our `finite state machine` to give the `AI-controlled agent` the `decision-making` logic necessary to switch between behaviors as desired.

---

Run the following command in terminal to create the 3 `states` of our `state machine`:

```bash
psyanim asset:fsmstate MyPatrolState MyIdleState MyFleeState -o ./src/patrolfsm
```

You should see 3 source files show up under the `/src/patrolfsm` directory - one for each state.

If you open up `MyIdleState.js`, you'll see that the state is a `javascript class` with a `contructor` and `3 methods`: `enter`, `exit`, and `run`.

The `constructor` is only executed once when the state is first created to be added to the state machine.

The `enter()` method is executed once every time the entity `enters` that state.

The `exit()` method is executed once every time the entity `exits` that state.

The `run(t, dt)` method is executed once every simulation frame, so long as the state is `active` (meaning the entity is currently in that state).

All `state transitions` should be added in the `constructor` of the state class, since we only want to add these transitions *once* for each state machine.

Every state machine maintains `state variables` that can be read / written via a set of APIs in the `PsyanimFSMState`.  These `state variables` can be used to trigger transitions.

The general workflow for implementing a `PsyanimFSMState` is as follows:

- Create the state using `psyanim-cli`
- Add transition conditions based on `state-variable` values in the `constructor` of the state class
- In `enter`, `exit`, and `run` methods, do any work and update `state variables` of the `state machine` as necessary, triggering state transitions when appropriate.

---

Go ahead and copy the raw contents of the finished source files for each state into your state files.  The links are:

- [MyIdleState.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/MyIdleState.js)
- [MyFleeState.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/MyFleeState.js)
- [MyPatrolState.js](https://github.com/thefinnlab/hello-psyanim2/blob/artificial_intelligence/src/MyPatrolState.js)

Let's look at `MyIdleState.js` in greater detail.  At the end of the `constructor` you can see the transitions to the `MyFleeState` and `MyPatrolState` are added.

Each call to `addTransition()` accepts 3 arguments:

- The `type` of the state to transition to.
- The name of the `state variable` the transition is based on
- The `condition` on which the value of the variable will `trigger` a `transition`

Next, take a look at the `enter()` and `run()` methods of our `MyIdleState` class.

Notice the enter() method just sets a `state variable` and sets up some references to be used later in `run()`.

In the `run()` method, a timer is updated which may or may not trigger a transition back to the `MyPatrolState`.  There is also a check to determine if the `agent` should transition to the `MyFleeState` based on proximity to the `player-controlled entity`.

The `enter()` and `run()` methods of any state do not need to check `state variables` to `trigger` state transitions.  This happens automatically in the `PsyanimFSM` state machine.

If you have time, as an exercise, check out all 3 states' source code and try to understand how the code there relates to what we see in our state machine diagram sketch from earlier.

---

The last thing we need to do is just create the `PsyanimFSM` component on the `agent` entity in our scene and add our states to it!

To do this, we'll use create a `PsyanimPatrolFleeAgentFSM` component that enapsulates the state machine creation.  Back in the terminal, run:

```bash
psyanim asset:component MyPatrolFleeAgentFSM -o ./src/patrolfsm
```

Open up the `MyPatrolFleeAgentFSM.js` and copy the following contents into it:

```js
import { 
    PsyanimComponent,
    
    PsyanimFSM,

} from 'psyanim2';

import MyFleeState from './MyFleeState.js';
import MyPatrolState from './MyPatrolState.js';
import MyIdleState from './MyIdleState.js';

export default class MyPatrolFleeAgentFSM extends PsyanimComponent {

    constructor(entity) {

        super(entity);

        this._fsm = this.entity.addComponent(PsyanimFSM);

        this._patrolState = this._fsm.addState(MyPatrolState);
        this._fleeState = this._fsm.addState(MyFleeState);
        this._idleState = this._fsm.addState(MyIdleState);

        this._fsm.initialState = this._patrolState;
    }
}
```

All this component does is add the `PsyanimFSM` to the `agent` entity and then add our 3 states to the `this._fsm` state machine.

Then, the last crucial step is to set the `initial state` of the `state machine` to the `MyPatrolState`.

Let's add this component to our `HelloAIScene` definition by first importing it at the top of the file and adding this `component definition` to the `agent` entity:

```js
{
    type: MyPatrolFleeAgentFSM,
}
```

Reload your `experiment` in the `browser` and you should see the `agent` entity `patrolling`, `fleeing` when you approach it too closely, and then waiting in an `idle state` for some time before attempting to return to a `patrol`.

Open your `browser console` to see that the state machine is displaying all the states and transitions automagically for you - this debug information didn't require any extra code or boilerplate as it's all part of the `PsyanimFSM` class out-of-the-box.

As you interact with the `AI agent` using your `player-controlled agent`, check to see that these `transitions` logged in the `browser console` make sense to you.

---

In this tutorial, we've created an `AI agent` for an `interactive experiment` that uses a `finite state machine` to decide which steering behaviors to execute at any given moment.

This is only a simple example of what's possible with the `Psyanim Decision-Making Framework` and `steering behavior` library.

The next step is to design your own `agent` behaviors! Try to create an `agent` that aggressively seeks the `player-controlled entity` when it gets too close, and then gives up after a certain distance from it's patrol path.

Think about how you could do this with a `PsyanimFSM` that executes an `arrive behavior` on the `player-controlled entity` in a certain `state` and what the `transition conditions` to return to patrolling might look like.

Remember, you can always return to the resources listed in the `references` section above as needed for more advanced study, too.