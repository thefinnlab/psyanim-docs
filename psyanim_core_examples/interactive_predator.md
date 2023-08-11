# Interactive Predator

In this example, we will create a scene with a single predator agent and a human-controlled agent.

The basic behavior of the predator is that it will wander about in the world until it detects another agent in it's *cone-shaped* field of view, at which point it will begin to chase that agent with a configurable degree of subtlety.

If the target agent leaves the predator's field of view, or if the chase goes on beyond a configurable duration, the predator returns to wandering around.

To see this in action, let's create a test scene using Psyanim-CLI:

```bash
npx psyanim-cli -s InteractivePredator
```

This scene will have several components it depends on, so let's open up the `InteractivePredator.js` scene under the `./src` directory and replace the `psyanim2` import line with the following:

```js
import 
{ 
    PsyanimScene,
    PsyanimConstants,
    PsyanimPlayerController,
    PsyanimArriveBehavior,
    PsyanimSeekBehavior,
    PsyanimWanderBehavior,
    PsyanimFOVSensor,
    PsyanimBasicPredatorBehavior,
    PsyanimPredatorAgent,
    PsyanimVehicle

} from 'psyanim2';
```

We can delete the `init()`, `preload()`, and `update()` methods from our `InteractivePredator` scene.

First, we'll add a player entity to the scene with a player controller component by adding the following lines to the end of our `create()` method as follows:

```js
create() {
    
    super.create();

    // create player
    this._player = this.addEntity('player', 400, 300, {
        shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
        radius: 12,
        color: 0x0000ff
    });

    let playerController = this._player.addComponent(PsyanimPlayerController);
    playerController.speed = 10;
}
```

At this point, you should be able to see your `player` entity in the scene and control it's movement using the WASD keys on your keyboard.

To setup a predator agent in the scene with the `player` entity as it's target, we will need a `PsyanimPredatorAgent`, which depends on the `PsyanimVehicle` component and the `PsyanimBasicPredatorBehavior` component.

That said, the `PsyanimBasicPredatorBehavior` has its own dependencies as it is composed of a `PsyanimArriveBehavior`, `PsyanimWanderBehavior`, and a `PsyanimFOVSensor`.

So, there will be a few different components we will need to setup / configure before we add our `PsyanimPredatorAgent` and `PsyanimBasicPredatorBehavior` to the `predator` entity.

To fully setup a `predator` agent, add the following code to the end of your scene's `create()` method:

```js
// setup predator agent
this._predator = this.addEntity('predator', 100, 100, {
    shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
    radius: 12, color: 0xffff00
});

let vehicle = this._predator.addComponent(PsyanimVehicle);

let arrive = this._predator.addComponent(PsyanimArriveBehavior);
arrive.maxSpeed = 6;
arrive.maxAcceleration = 0.2;
arrive.innerDecelerationRadius = 16;
arrive.outerDecelerationRadius = 40;

let seek = this._predator.addComponent(PsyanimSeekBehavior);
seek.maxSpeed = 4;
seek.maxAcceleration = 0.2;

let wander = this._predator.addComponent(PsyanimWanderBehavior);
wander.seekBehavior = seek;
wander.radius = 50;
wander.offset = 250;
wander.maxWanderAngleChangePerFrame = 20;

let fovSensor = this._predator.addComponent(PsyanimFOVSensor);

let predator = this._predator.addComponent(PsyanimBasicPredatorBehavior);
predator.arriveBehavior = arrive;
predator.wanderBehavior = wander;
predator.fovSensor = fovSensor;
predator.subtlety = 30;
predator.subtletyLag = 500;

let predatorAgent = this._predator.addComponent(PsyanimPredatorAgent);
predatorAgent.vehicle = vehicle;
predatorAgent.predatorBehavior = predator;
predatorAgent.target = this._player;
```

Finally, to disable screen boundary wrapping and keep the agents inside the canvas at all times, add the following line at the end of your `create()` method:

```js
this.screenBoundary.wrap = false;
```

You should now have a working `Interactive Predator` scene!  You can tune the parameters to get more nuanced, desired results.

The full code for our `InteractivePredator.js` can be found [here](https://github.com/thefinnlab/psyanim-core-examples/blob/master/src/InteractivePredator.js).