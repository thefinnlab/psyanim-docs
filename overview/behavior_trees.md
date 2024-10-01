# <ins>AI Behavior Trees</ins>

## 1. Core Concepts
In the previous section, we used `finite state machines` for `agent decision-making.`

In this section, we'll introduce a more flexbile, and still very powerful, agent decision-making structure known as `AI Behavior Trees`.

---

`Behavior trees`, as the name implies, are a tree structure that encode logic for agent decision-making.

Behavior trees are composed of connected nodes, which are either `composites` or `tasks`:

- `Composites` are nodes that have other nodes as children.

- `Tasks` are leaf nodes in the tree which perform specific actions.

Each node can contain any number of `decorators` which are evaluated to determine whether or not a node can run.

---

Behavior tree task and decorators communicate with `psyanim components` via an in-memory data structure known as a `blackboard`.

The `blackboard` also serves as a mechanism for other external code, including the `PsyanimJsPsychTrial` API, to communicate with behavior tree tasks and decorators.

A major advantage of behavior trees is the loose coupling between `tasks` (as opposed to the tight coupling between `states` in an `FSM`).

Another major advantage of behavior trees is how easy it is to visualize and manipulate using visual editing tools, compared to state machines.

For a general overview of behavior trees, I highly recommend you check out the video series by Petter Ögren:

- [5 minute Behavior Tree tutorial](https://youtu.be/KeShMInMjro?feature=shared)

- [Introduction to Behavior Trees Series](https://www.youtube.com/playlist?list=PLFQdM4LOGDr_vYJuo8YTRcmv3FrwczdKg)

---

`psyanim-3` has both a `Behavior Tree Engine` as well as a graphical `Behavior Tree Editor` called `Psyanim Behavior Designer` that runs cross-platform in the browser.

The `Behavior Designer` is a visual tool for used to build a behavior tree definition, which contains the logic for all decision-making associated with a particular agent.

The Psyanim Behavior Tree Engine and Behavior Designer are heavily inspired by, but not identical to, the Behavior Tree system in Unreal Engine 5.

To get a good feel for the basics of Behavior Trees in general, the first 7 min. and 15 seconds of the following video is a worthwhile watch:

- [Unreal Engine AI With Behavior Trees](https://youtu.be/iY1jnFvHgbE?feature=shared)

## 2. Hands-on Video Walkthrough

While there certainly are universal behavior tree concepts across implementations, there is not a strong agreement about precise definitions and features considered core to a tree.

Every implementation can (and often does) vary significantly from others, thus leaving it up to the AI designers to become familiar with the specific tool they are using.

With that in mind, here is a video that walks you through the usage of `psyanim-3's Behavior Designer` usage specifically in the context of a new `psyanim-3 experiment project`:

<p align="center" style="font-size: 12px;">
    <video width="640" height="360" controls>
    <source src="./videos/behavior-designer-tutorial.mp4" type="video/mp4">
    </video>
</p>

## 3. Behavior Designer Cheat-sheet

Here's a quick and dirty list of controls for behavior designer, (but you can see them all used in the video in the previous section!):

- Right-click or press `space bar` in the graph editor canvas area to open the context menu with available nodes

- Clicking on a node in the node editor brings up its properties in the node editor.

- `Delete` key on keyboard deletes selected nodes.

- Deleting a node automatically deletes associated edges

- Hovering over an edge and holding the `Alt` key while left-clicking will delete an edge in the graph editor.

- Left-click and drag to select multiple nodes in the graph editor.

- All of the fields in the node editor are collapsible by clicking on the field name.

- When a node has multiple decorators, you can select one and drag it up and down to re-order the decorators on a node. Decorators will be executed from top to bottom.

- The number on the top-right hand side of a node or decorator is the `node index` and determines it's execution order relative to other nodes or decorators.

- `File->Save` to save the document using the current document name. By default, the current document is named `NewBehaviorTree`.

- `File->Save As` to save the document using a different name.  Using `File->Save` after this will always save to the most recently saved document using `Save As` (unless you reload the page).

- `File->Load` allows you to load an existing behavior tree file.