# Psyanim 2.0: jsPsych Integration Tutorial

Note: Full tutorial project + source code can be found [here](https://TODO-brokenlink).

## 1. Psyanim-jsPsych Integration

`Psyanim 2.0` is designed as a standalone [Phaser](https://phaser.io/) app which renders to an `HTML Canvas`, as any other `Phaser` app does.

In order to use `Psyanim 2.0` in [jsPsych](https://www.jspsych.org/), a `jsPsych plugin` was developed: `PsyanimJsPsychPlugin`.

In this tutorial, we'll walk through how to use a `Psyanim scene` as a `jsPsych trial`, as well as how to create variations of the same scene in different trials, each with their own independent sets of parameters for the same scene.

## 2. The Anatomy of the `PsyanimJsPsychPlugin`

The main learning curve in `Psyanim 2.0` is understanding how to build interactive content in the `Psyanim scene definitions` and extending the framework with custom `Psyanim components`.

The `PsyanimJsPsychPlugin` is designed to allow you to easily run `Psyanim Scenes` as `jsPsych trials`.

---

Let's take a moment to walk through some of the fundamentals of using the `PsyanimJsPsychPlugin` in a `jsPsych experiment`.

For starters, note that any `Psyanim scene definition` must be registered with `PsyanimApp.Instance.config.registerScene()` before it can be referenced by `scene key` from a `jsPsych trial`.

Also note how we set the `PsyanimApp canvas` to be invisible at the start of the experiment.  This is desirable in most cases, as we may not have a `Psyanim scene` as the first jsPsych trial.

All of the `jsPsych` usage is the same as any other experiment, as described in the [docs](https://www.jspsych.org/).

The only main structural change we've made from our previous `tutorial` is that our jsPsych `timeline` is now declared as a named variable and we are pushing trials into it before passing it to `jsPsych.run()` at the bottom of the file.

Adding a `jsPsych trial` using the `PsyanimJsPsychPlugin` is very simple, with only two fields in the trial definition object: 

- `type`: the standard `jsPsych` trial definition `type` field, which specifies the type of `jsPsych plugin` this trial will use
- `sceneKey`: this is the `scene key` for any `Psyanim scene definition` that's been registered with `PsyanimApp`

And that's all there really is to the `PsyanimJsPsychPlugin` interface!

It's not much more complex than using any other `jsPsych plugin`. 

## 3. Setting up our test project

To start, create a new empty Psyanim 2 project, following the same steps as the [Hello Psyanim 2.0 tutorial](/overview/hello_psyanim_2.md#2-creating-a-new-psyanim-2-project).

