# Introduction and Quick Start

`Psyanim 2.0` is a tool for rapid design, prototyping and deployment of web-based psychology research experiments involving 2D procedural game AI & animation.

In order to create experiments with different `Psyanim` scenes and agent behaviors, a basic understanding of Javascript is recommended, but not beyond what is required to use [jsPsych](https://www.jspsych.org/).

A basic understanding of [jsPsych](https://www.jspsych.org/) is also necessary.

To extend Psyanim with new functionality or AI behavior algorithms, an intermediate level of comfort with Javascript is recommended.

## Quick Start

***Pre-requisites: Requires [NodeJS](https://nodejs.org/en) v18.16.0+, an up-to-date installation of [Git](https://git-scm.com/), and [Psyanim-CLI](https://github.com/thefinnlab/psyanim-cli.git) installed globally.***

***You'll need to make sure you have read access to our psyanim-2 and psyanim-cli private git repos***

If not installed already, you can install psyanim-cli package globally from git repo via npm with the following command:

```bash
npm install -g git+https://github.com/thefinnlab/psyanim-cli.git
```

---

You can create a new `psyanim-2` experiment project from any empty directory with:

```bash
psyanim init
```

To start a watch service to rebundle your project any time a code file changes, use this command in a separate terminal:

```bash
npm run watch
```

To run a local development server on port `3000`, use the following command in a separate terminal:

```bash
npm run serve
```

---

The [Hello Psyanim 2.0](/overview/hello_psyanim_2.md) tutorial will familiarize you with the core components of `Psyanim 2.0`, as well as the project structure and command-line interface tool, Psyanim-CLI.