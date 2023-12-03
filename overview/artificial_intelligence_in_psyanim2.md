# Artificial Intelligence in Psyanim-2

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

## 3. AI Steering Behaviors and Agents in Psyanim 2

TODO: this is a WIP still!


Let's get started by creating a new, empty `psyanim2` project using `psyanim-cli` as we did in the [Hello Psyanim 2.0 tutorial](/overview/hello_psyanim_2.md#2-creating-a-new-psyanim-2-project).

You can delete the `EmptyScene.js` and remove all the commented out code from the `index.js`. 

Let's create an new scene with `psyanim-cli`:

```bash
psyanim -s HelloAIScene
```



## 4. Finite State Machines in Psyanim 2

TODO: this is a WIP still!

One big advantage of a state machine is that it can be easily visualized and reasoned about using simple boxes and arrows.

When designing a state machine for any purpose, I always start with a pencil and a piece of paper, sketching out the `states` represented by `boxes` and `allowed transitions` represented by `arrows`.

I highly recommend that, before trying to code up any state machine, you start by sketching out a simple diagram of it's design to make it easy to reason about as you implement it.

<p align="center">
  <img src="./imgs/ai_fsm_design.jpg" />
</p>

refs: https://gameprogrammingpatterns.com/state.html