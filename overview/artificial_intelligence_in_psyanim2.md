# Artificial Intelligence in Psyanim-2

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

The `AI agent` will `patrol` back and forth between a few points in the world, and when approached by the `player-controlled agent`, the `AI agent` will `flee` from it, waiting a bit some distance away before attempting to return back to it's `patrol`.

We will give the AI agent a 'brain' later to decide how to switch between behaviors (known as `decision-making` in game AI), but for this section, let's just get the AI agent patrolling between a few points using a `pathfollowing` steering behavior, so we can learn a bit about the `steering architecture` in general.

Let's get started by creating a new, empty `psyanim2` project using `psyanim-cli` as we did in the [Hello Psyanim 2.0 tutorial](/overview/hello_psyanim_2.md#2-creating-a-new-psyanim-2-project).

Let's create an new scene with `psyanim-cli`:

```bash
psyanim -s HelloAIScene
```

You can delete the `EmptyScene.js`, remove all the commented out code from the `index.js` and add a trial for `HelloAIScene.js` to it.

---

Open `HelloAIScene.js` and let's start building out our scene.  We can delete the `navgrid` property and set the `wrapScreenBoundary` property to `true`.

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

The `PsyanimArriveBehavior` component is responsible for calculating `steering forces` for an `entity`.

The `PsyanimArriveBehavior` is *not* responsible applying these `steering forces` each frame.  

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

This separation of concerns allows for `*Behavior` components to be *reused* in other `*Behavior`s or `*Agents`.

## 4. Finite State Machines in Psyanim 2

TODO: this is a WIP still!

One big advantage of a state machine is that it can be easily visualized and reasoned about using simple boxes and arrows.

When designing a state machine for any purpose, I always start with a pencil and a piece of paper, sketching out the `states` represented by `boxes` and `allowed transitions` represented by `arrows`.

I highly recommend that, before trying to code up any state machine, you start by sketching out a simple diagram of it's design to make it easy to reason about as you implement it.

<p align="center">
  <img src="./imgs/ai_fsm_design.jpg" />
</p>

refs: https://gameprogrammingpatterns.com/state.html