# <ins>Getting Started with Psyanim 2.0</ins>

## 1. Introduction and Installation

***Pre-requisites: Requires [NodeJS](https://nodejs.org/en) v18.16.0+, an up-to-date installation of [Git](https://git-scm.com/), and [Psyanim-CLI](https://github.com/thefinnlab/psyanim-cli.git) installed globally.***

***You'll need to make sure you have `read` access to our `psyanim-2` and `psyanim-cli` private git repos***

`Psyanim 2.0` is a framework for creating browser-based psychology / cognitive science research experiments involving 2D procedural animation.

`Psyanim-CLI` is a tool for creating & managing experiment projects and assets in `Psyanim 2.0`.

If not installed already, you can install psyanim-cli package globally from git repo via npm with the following command:

```bash
npm install -g git+https://github.com/thefinnlab/psyanim-cli.git
```

## 2. Creating A New Project & First Scene

In a terminal, create a directory called `hello-psyanim2` and run the following command from that directory:

```bash
psyanim init
```

This command will create a blank npm project, install psyanim-2 & it's dependencies, and then setup a local git repo to keep things under version control.

Let's create a new scene by running the following command:

```bash
psyanim template:predpreyv2 -o ./src/scenes
```

This command creates a new type of experiment `asset` called a `scene definition` under `./src/scenes/HelloPredatorPreyV2.js`.

This particular scene definition is a `template` which already has two `agents` setup, where one behaves as a `predator` and the other behaves as a `prey`.

Let's add this scene definition to our `jsPsych experiment` now.

Open up `index.js` under the `./src` directory and add the following import after the rest of the imports:

```js
import PredatorPreyV2 from './scenes/PredatorPreyV2.js';
```

You can delete the `EmptyScene` import statement before that.

Scroll down to the line where we create a variable called `emptySceneTrial` and delete that whole trial, including the line where it's pushed into the `jsPsych timeline`.

Next, in the location where the `emptySceneTrial` previously was, we'll add our `PredatorPreyV2` scene definition to a trial in our jsPsych timeline with the following code:

```js
let predatorPreyV2Trial = new PsyanimJsPsychTrial(PredatorPreyV2, PredatorPreyV2.key);

timeline.push(predatorPreyV2Trial);
```



## 3. Core Concepts

