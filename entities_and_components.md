# Entities and Components

## NOTE: this is still a work-in-progress and missing key explanations in certain areas.  Also will be making more concise by moving finished class defs to a separate git repo.  Will be completed soon tho!

## 1. What's a component?

To this point, we've learned that `scenes` are like a stage where characters and animations are presented to the user, and `entities` are *things* that live in `scenes`.

More specifically, `entities` are any objects in a scene that have a position, orientation and velocity which can be controlled either directly or via forces / accelerations.

They can also, optionally, have a visual representation in the scene.

In the previous tutorial, we were able to add logic to our scene which is executed every frame to move an entity (our 'player' entity) around in response to control input from the keyboard.

While adding logic directly to our scene is a powerful ability, it isn't easily portable to other scenes or entities.  

We certainly don't want scenes referencing other scenes, b.c. then all of our scenes would be tightly coupled to each other... and since experiments are made up of scenes, our experiments could become tightly coupled to each other.

The solution to this problem is what we call a `Psyanim Component`.

A `PsyanimComponent` is a modular object that can be attached to an entity, and which has it's own user-defined `update(t, dt)` method which runs every simulation frame.

`PsyanimComponents` can be attached to any entity and any scene, thus making it easy to reuse and a better place for most of our AI & animation logic than directly in the scene itself.

As a general rule of thumb, if you have logic that is specific to one scene only and doesn't need to be reusable across scenes or different entities, it's probably OK to put that logic directly into the `PsyanimScene` definition itself.

If you do need to reuse a bit of behavior or game logic across multiple entities or scenes, it's best to abstract that out into a `PsyanimComponent`.

In the following tutorial, we'll explore `PsyanimComponents` in depth.

## 2. Components in action: creating a simple game in `Psyanim 2.0` with multiple levels

To build a more intuitive feel for the power of `PsyanimComponents` and how best to utilize them, we can create a simple game consisting of 3 levels.

The 'levels' will be implemented as `PsyanimScenes`.

In this game, we'll build a `Player Controller` component that can be reused across scenes on different entities.

## 3. Project Setup: Setup your npm project and install psyanim-2

Let's start by opening a terminal (or powershell if that's your thing :) and creating a directory called `hello-psyanim-components`.

Navigate to the directory we just created and initialize an npm project with:

```bash
    npm init -y
```

Install psyanim-2 and psyanim-cli with:

```bash
    npm install git+https://github.com/thefinnlab/psyanim-2.git git+https://github.com/thefinnlab/psyanim-cli.git
```

Now, we can initialize our psyanim experiment with:

```bash
    npx psyanim-cli --init
```

Finally, navigate to the `hello-psyanim-components` directory in your browser and start a webpack watch service with:

```bash
    npm run watch
```

Now you can start a static file server of your choice to serve up the files in your `./dist/` directory.

## 4. Level Creation: Setup three 'levels' as a scenes

Let's go ahead and delete the `EmptyScene.js` under `./src/`.

We can create the our scenes for our 'levels' with the following command:

```bash
    npx psyanim-cli -s MyLevel1,MyLevel2,MyLevel3
```

You should see the scene files for those levels show up under `./src/`.

Then, open up `index.js` in the `./src/` directory.

We won't need `jsPsych` here, so go ahead and delete everything in your index.js and we'll build it from scratch as an exercise.

Copy the following code into your `index.js` file:

```js
    import { PsyanimApp } from 'psyanim2';

    import MyLevel1 from './MyLevel1';
    import MyLevel2 from './MyLevel2';
    import MyLevel3 from './MyLevel3';

    /**
     *  Setup Psyanim and PsyanimJsPsychPlugin
     */
    PsyanimApp.Instance.config.registerScene(MyLevel1);
    PsyanimApp.Instance.config.registerScene(MyLevel2);
    PsyanimApp.Instance.config.registerScene(MyLevel3);

    PsyanimApp.Instance.run();
```

Open up each level's scene file under `./src/` and delete the `init()`, `preload()`, and `update(t, dt)` methods in each one.

We will be putting all of our game logic & behaviors into `PsyanimComponents` we'll be creating, so the only method we'll need in our scene is `create()`, which is where we'll declare all of our entities and attach components to them.

## 5. Phaser APIs: Using Phaser 3 inside of Psyanim

There are only *four* very simple relationships you'll need to remember to access *all* `Phaser 3` functionality from within a `PsyanimComponent`:

- A `PsyanimScene` *is* a `Phaser.Scene`.
- A `PsyanimEntity` *is* a `Phaser.Physics.Matter.Sprite`.
- A `PsyanimComponent` has a reference to the `PsyanimEntity` to which it is attached.
- A `PsyanimEntity` has a reference to the `PsyanimScene` in which it exists.

Thus, we can access any `Phaser.Scene` property or method via a `PsyanimScene` reference and can use any of the Phaser APIs anywhere in a `PsyanimScene` or `PsyanimComponent`.

Moreover, we can access the `entity` a `component` is attached to from anywhere inside the `component`, and from the `entity` reference, we can access the `PsyanimScene` it lives in.

## 6. Our first component: A level loader

In this section, we'll explore how `PsyanimScenes`, `PsyanimEntities` and `PsyanimComponents` relate to and interact with the `Phaser 3` game engine by building a custom level loader component.

Right now, our game has 3 levels, but the first level loaded on app start is the first scene we registered with `PsyanimApp`, which is `MyLevel1`.

Once our `MyLevel1` scene loads, it will remain the active scene in our game until we load a different level from our code.

Since we want to be able to load a level from any scene, it's not ideal to copy / paste our level loading code into every single scene we create.

Instead, let's create a `PsyanimComponent`, called `MyLevelLoader`, which can be added to any scene which allows us to load a level via a keypress.

To create the component in our project, simply navigate to our project directory in your terminal and run:

```bash
    npx psyanim-cli --component MyLevelLoader
```

Open up `MyLevelLoader.js` under `./src/` and you'll see the core of a `PsyanimComponent` class is very minimal, fundamentally only requiring a `constructor()` and an `update(t, dt)` method in its interface.

Add the following code to the constructor so it looks like this:

```js
constructor(entity) {

    super(entity);

    let scenePlugin = this.entity.scene.scene;

    console.log(scenePlugin.key + " loaded!");

    this._sceneKeys = Object.keys(scenePlugin.manager.keys);

    this._keys = {
        alpha1: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.ONE),
        alpha2: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.TWO),
        alpha3: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.THREE)
    };
}
```

Notice here that from the constructor of the `MyLevelLoader` component, we are able to get a reference to the entity it's attached to with `this.entity`.

From the entity reference, we're able to get a reference to the `PsyanimScene` it's in with `this.entity.scene`.

Since the `PsyanimScene` *is* a `Phaser.Scene`, we're able to use any of the [Phaser APIs](https://newdocs.phaser.io/docs/3.60.0/Phaser.Scene) on the `PsyanimScene` reference.

From there, we grab a reference to the Phaser `scenePlugin` and that gives us access to both the current scene's `key` (unique identifier string) as well as methods to pause the current scene, load a different scene, etc.

In the constructor, we store a reference to all the available scene keys for later use and then we setup some control inputs for switching scenes by pressing the '1', '2', or '3' keys on the keyboard.

Modify your `update(t, dt)` method of your `MyLevelLoader` component so it looks like the following:

```js
update(t, dt) {
    
    super.update(t, dt);

    let scenePlugin = this.entity.scene.scene;

    if (Phaser.Input.Keyboard.JustDown(this._keys.alpha1))
    {
        scenePlugin.start(this._sceneKeys[0]);
    }
    else if (Phaser.Input.Keyboard.JustDown(this._keys.alpha2))
    {
        scenePlugin.start(this._sceneKeys[1]);
    }
    else if (Phaser.Input.Keyboard.JustDown(this._keys.alpha3))
    {
        scenePlugin.start(this._sceneKeys[2]);            
    }
}
```

Here, again, we access the entity to which this component is attached via `this.entity` and then we get a reference to the current scene via `this.entity.scene`

We then grab the scene plugin reference via `this.entity.scene.scene`, as per the [Phaser.Scene](https://newdocs.phaser.io/docs/3.60.0/Phaser.Scene) APIs.

Finally, we check to see if any of the keys we setup were pressed, and if so, we load the corresponding scene (`MyLevel1`, `MyLevel2`, or `MyLevel3`).

The full source for your `MyLevelLoader` component should look like the following:

```js
import Phaser from 'phaser';

import { PsyanimComponent } from 'psyanim2';

export default class MyLevelLoader extends PsyanimComponent {

    constructor(entity) {

        super(entity);

        let scenePlugin = this.entity.scene.scene;

        console.log(scenePlugin.key + " loaded!");

        this._sceneKeys = Object.keys(scenePlugin.manager.keys);

        this._keys = {
            alpha1: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.ONE),
            alpha2: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.TWO),
            alpha3: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.THREE)
        };
    }

    update(t, dt) {
        
        super.update(t, dt);

        let scenePlugin = this.entity.scene.scene;

        if (Phaser.Input.Keyboard.JustDown(this._keys.alpha1))
        {
            scenePlugin.start(this._sceneKeys[0]);
        }
        else if (Phaser.Input.Keyboard.JustDown(this._keys.alpha2))
        {
            scenePlugin.start(this._sceneKeys[1]);
        }
        else if (Phaser.Input.Keyboard.JustDown(this._keys.alpha3))
        {
            scenePlugin.start(this._sceneKeys[2]);            
        }
    }
}
```

Great work!  If you reload the app in your browser, you should now be able to switch scenes with the '1', '2', or '3' keys on your keyboard! 

(check the Chrome Dev Tools console output to see what scene is currently loaded)

## 7. Player Entities: Adding our player entities to our levels

Now let's explore adding entities to our levels.  Note that we're using `levels` and `scenes` interchangeably here because each level is implemented as a `PsyanimScene`.

Open up each of your level scene files and modify the `psyanim-2` import statement so it also imports `PsyanimConstants`:

```js
import 
{ 
    PsyanimScene,
    PsyanimConstants

} from 'psyanim2';
```

Next, in the first level, let's add a green circle-shaped player by calling `addEntity` in our scene class's `create()` as follows:

```js
    create() {

        super.create();

        let sceneControls = this.addEntity('sceneControls');
        sceneControls.addComponent(MyLevelLoader);

        this.addEntity('player', 400, 300, {
            shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE,
            radius: 12, color: 0x00ff00
        });
    }
```

In the second level, let's add a yellow rectangle-shaped player:

```js
create() {

    super.create();

    let sceneControls = this.addEntity('sceneControls');
    sceneControls.addComponent(MyLevelLoader);

    this.addEntity('player', 400, 300, {
        shapeType: PsyanimConstants.SHAPE_TYPE.RECTANGLE,
        width: 50, height: 30, color: 0xffff00
    });
}
```

In the third level, let's add a blue triangle-shaped player:

```js
create() {

    super.create();

    let sceneControls = this.addEntity('sceneControls');
    sceneControls.addComponent(MyLevelLoader);

    this.addEntity('player', 400, 300, {
        shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE,
        base: 15, altitude: 30, color: 0x0000ff
    });
}
```

Reload your app in your browser and you should now see each scene has a different shape and color for the player!

## 8. Player Controller Component: creating a player controller component that can be reused across levels

Let's create a new component, `MyPlayerContorller`, using Psyanim CLI in terminal:

```bash
    npx psyanim-cli -c MyPlayerController
```

Open up `MyPlayerController.js` under `./src/` and replace its contents with the following code:

```js
import Phaser from 'phaser';

import { PsyanimComponent } from 'psyanim2';

export default class MyPlayerController extends PsyanimComponent {

    // define translational and rotational speeds
    speed;
    turnSpeed;

    constructor(entity) {

        super(entity);

        this.speed = 8;
        this.turnSpeed = 0.2;

        // setup keyboard controls for this scene
        this._keys = {
            W: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.W),
            A: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.A),
            S: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.S),
            D: this.entity.scene.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.D)
        };
    }

    update(t, dt) {
        
        super.update(t, dt);

        // get horizontal and vertical movement inputs
        let horizontal = (this._keys.A.isDown ? -1 : 0) + (this._keys.D.isDown ? 1 : 0);
        let vertical = (this._keys.W.isDown ? -1 : 0) + (this._keys.S.isDown ? 1 : 0);

        // compute velocity direction from keyboard inputs and set to desired speed
        let velocity = new Phaser.Math.Vector2(horizontal, vertical)
            .setLength(this.speed);

        // here we actually set the translational velocity of the 'player' entity:
        this.entity.setVelocity(velocity.x, velocity.y);

        // next we will adjust our orientation based on our control input directions
        if (Math.abs(horizontal) > 1e-3 || Math.abs(vertical) > 1e-3)
        {
            const targetAngle = Math.atan2(vertical, horizontal);

            // we don't snap to the target angle immediately... 
            // instead we lerp smoothly to it using Phaser's built-in 'RotateTo' method
            let newAngle = Phaser.Math.Angle.RotateTo(
                this.entity.angle * Math.PI / 180,
                targetAngle,
                this.turnSpeed);

            // here, we actually set the player's orientation based on the lerped value
            this.entity.setAngle(newAngle * 180 / Math.PI);
        }
    }
}
```

The details of the movement algorithm have been explained in the previous tutorial on `Scenes And Entities`, so we won't go over those here.

What is of interest to us in `MyPlayerController` is that `speed` and `turnSpeed` are now properties of the class.

Also, note that the `setVelocity()` and `setAngle()` methods are called on `this.entity` now, because this component should work for any `PsyanimEntity` it's attached to.

## 9. Object Interaction: Creating some collectibles or pickups

### TODO: this needs to be finished still

Let's create a `MyCollectibleItem` component using Psyanim CLI:

```bash
    npx psyanim-cli -c MyCollectibleItem
```

Open up `MyCollectibleItem.js` and replace its contents with the following code:

```js
import Phaser from 'phaser';

import { PsyanimComponent } from 'psyanim2';

export default class MyCollectibleItem extends PsyanimComponent {

    player;

    constructor(entity) {

        super(entity);

        this.entity.setOnCollide(this._handleCollision.bind(this));
    }

    _handleCollision(matterCollisionData) {

        let collidedWithPlayer = false;

        // check bodyA and bodyB to see if either one belongs to the 'player' entity
        if (matterCollisionData.bodyA.gameObject)
        {
            collidedWithPlayer = matterCollisionData.bodyA.gameObject.name == 'player';
        }
        else if (matterCollisionData.bodyB.gameObject)
        {
            collidedWithPlayer = matterCollisionData.bodyB.gameObject.name == 'player';
        }

        if (collidedWithPlayer)
        {
            console.log('collectible picked up!');
            this.entity.destroy();
        }
    }
}
```

Now open up `MyLevel1.js`, `MyLevel2.js`, and `MyLevel3.js` and add the following import at the top of each file:

```js
    import MyCollectibleItem from './MyCollectibleItem';
```

In `MyLevel1.js` create an `entity` for our `collectible item` and add a `MyCollectibleItem` component to it:

```js
    // add a single collectible
    let collectible = this.addEntity('collectible', 600, 500, {
        shapeType: PsyanimConstants.SHAPE_TYPE.CIRCLE, 
        radius: 5, color: 0xbf40bf,
        textureKey: 'purple_collectible'
    },
    {
        isSensor: true
    });

    collectible.addComponent(MyCollectibleItem);
```

Notice the `isSensor` field of the last object parameter in our `addEntity()` call.  When this is set to true, we can still receive collision events for this entity, but it will not impart or receive any impulses.

Also notice the `textureKey` field in the second to last object parameter.  Whenever we create an `entity` with a particular shape, a texture is generated for it with a particular key.  We can override this key's value by setting it in the `textureKey` field.

If we supply a `textureKey` that's already got a texture generated for it, the entity will use that texture instead of generating a new one.  This is a helpful optimization if you have a lot of entities with the same shape, size and color.

Now that we've added one collectible to our `MyLevel1` scene, reload the app in your browser and you should be able to move your player character over to it.

The `collectible` should disappear when you touch it and the Chrome Debug Tools will show 'collectible picked up!'.

Open `MyLevel2.js` and add the following code at the end of the `create()` method:

```js
        // setup collectibles
        let collectibleSpawnPoints = [
            { x: 100, y: 100 },
            { x: 100, y: 500 },
            { x: 700, y: 100 },
            { x: 700, y: 500 },
        ];

        for (let i = 0; i < collectibleSpawnPoints.length; ++i)
        {
            let collectible = this.addEntity('collectible' + i, 
            collectibleSpawnPoints[i].x, collectibleSpawnPoints[i].y, {
                shapeType: PsyanimConstants.SHAPE_TYPE.RECTANGLE, 
                width: 8, height: 8, color: 0x0000ff,
                textureKey: 'blue_collectible'
            },
            {
                isSensor: true
            });
    
            collectible.setAngle(45);

            collectible.addComponent(MyCollectibleItem);    
        }
```

Here, we add 4 blue diamond-shaped `collectibles` by creating 4 collectible entities with a blue square shape and then rotating the squares 45 degrees.

Notice that we were able to define a set of spawn points for the collectibles in pixel coordinates and then call `this.addEntity()` in a loop to add collectibles to the scene at that point.

Also notice that we use the same `textureKey` value for each collectible, as an optimization, since they all can share the same texture.

Reload the app in your browser and hit the '2' key on your keyboard to load up scene 2 and you should see 4 diamond-shaped collectibles that disappear when you touch them, leaving a note in console that they were picked up.

We'll add some collectibles to `MyLevel3`, but to make it more interesting, let's make these collectibles move!

## 10. Modularity by Design: Creating a tweening component

In this section, we'll see how to use Phaser's Tween functionality from a PsyanimComponent, in turn getting more practice using Phaser 3's powerful feature-set from within a PsyanimComponent!

```bash
    npx psyanim-cli -c MyTweener
```

```js
import Phaser from 'phaser';

import { PsyanimComponent } from 'psyanim2';

export default class MyTweener extends PsyanimComponent {

    point1;
    point2;

    duration;

    constructor(entity) {

        super(entity);

        this.duration = 3000;

        this._boundStartTween = this._startTween.bind(this);

        this.scene.events.on('create', this._boundStartTween);
    }

    destroy() {

        if (this._tween)
        {
            this._tween.stop();
            this._tween.destroy();
            this._tween = null;
        }

        if (this._boundStartTween)
        {
            // make sure we unsub from the scene's create callback
            this.scene.events.off('create', this._boundStartTween);
            this._boundStartTween = null;
        }

        super.destroy();
    }

    _startTween() {

        // set this entity's position to point1
        this.entity.position = this.point1;

        // set this entity to yoyo tween to point2
        this._tween = this.scene.add.tween({
            targets: this.entity,
            x: this.point2.x,
            y: this.point2.y,
            duration: this.duration,
            yoyo: true,
            ease: 'Linear',
            repeat: -1
        });
    }
}
```

## 11. Combining Multiple Components

```js
import Phaser from 'phaser';

import 
{ 
    PsyanimScene,
    PsyanimConstants

} from 'psyanim2';

import MyLevelLoader from './MyLevelLoader';
import MyPlayerController from './MyPlayerController';
import MyCollectibleItem from './MyCollectibleItem';
import MyTweener from './MyTweener';

export default class MyLevel3 extends PsyanimScene {

    static KEY = 'MyLevel3';

    constructor() {

        super(MyLevel3.KEY);
    }

    create() {

        super.create();

        let sceneControls = this.addEntity('sceneControls');
        sceneControls.addComponent(MyLevelLoader);

        let player = this.addEntity('player', 400, 300, {
            shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE,
            base: 15, altitude: 30, color: 0x0000ff
        });

        player.addComponent(MyPlayerController);

        // setup collectibles
        let tweenPoints = [
            [
                { x: 100, y: 100 },
                { x: 700, y: 100 }
            ],
            [
                { x: 700, y: 100 },
                { x: 700, y: 550 }
            ],
        ];

        for (let i = 0; i < tweenPoints.length; ++i)
        {
            let collectible = this.addEntity('collectible' + i, 
                tweenPoints[i].x, tweenPoints[i].y, {
                shapeType: PsyanimConstants.SHAPE_TYPE.TRIANGLE, 
                base: 24, altitude: 20, color: 0x00ff00,
                textureKey: 'green_collectible'
            },
            {
                isSensor: true
            });
    
            collectible.setAngle(-90);

            collectible.addComponent(MyCollectibleItem);    

            let tweener = collectible.addComponent(MyTweener);
            tweener.point1 = tweenPoints[i][0];
            tweener.point2 = tweenPoints[i][1];
        }
    }
}
```

## 12. Adding Game Logic: Creating a Game Manager

Here we add an event emitter to `MyCollectibleItem.js` for component communication.

`constructor`:

```js
    this.events = new Phaser.Events.EventEmitter();
```

`_handleCollision()`:

```js
    _handleCollision(matterCollisionData) {

        let collidedWithPlayer = false;

        // check bodyA and bodyB to see if either one belongs to the 'player' entity
        if (matterCollisionData.bodyA.gameObject)
        {
            collidedWithPlayer = matterCollisionData.bodyA.gameObject.name == 'player';
        }
        else if (matterCollisionData.bodyB.gameObject)
        {
            collidedWithPlayer = matterCollisionData.bodyB.gameObject.name == 'player';
        }

        if (collidedWithPlayer)
        {
            this.events.emit('collected', this);

            this.entity.destroy();
        }
    }
```

Here we implement the game manager component:

```bash
    npx psyanim-cli -c MyGameManager
```

```js
import Phaser from 'phaser';

import { PsyanimComponent } from 'psyanim2';

import MyCollectibleItem from './MyCollectibleItem';

export default class MyGameManager extends PsyanimComponent {

    constructor(entity) {

        super(entity);

        this._boundOnAfterSceneCreated = this._onAfterSceneCreated.bind(this);

        // we get the collectibles *after* scene::create() method is completed
        this.scene.events.on('create', this._boundOnAfterSceneCreated);
    }

    _onAfterSceneCreated() {

        this._collectibles = this.scene.getComponentsByType(MyCollectibleItem);

        this._collectibles.forEach(collectible => {

            collectible.events.on('collected', (c) => {

                this._removeCollectible(c);
                this._checkIfWonGame();
            });
        })
    }

    _removeCollectible(collectible) {

        let index = this._collectibles.indexOf(collectible);

        if (index >= 0)
        {
            this._collectibles.splice(index, 1);
        }
        else
        {
            console.error("Failed to find collectible to remove!");
        }
    }

    _checkIfWonGame() {

        if (this._collectibles.length == 0)
        {
            console.log('Great job - you found all the collectibles!');
        }
    }

    destroy() {

        if (this._boundOnAfterSceneCreated)
        {
            // unsub from the 
            this.scene.events.off('create', this._boundOnAfterSceneCreated);
            this._boundOnAfterSceneCreated = null;
        }

        super.destroy();
    }

    update(t, dt) {
        
        super.update(t, dt);
    }
}
```

In `MyLevel1.js`, `MyLevel2.js`, and `MyLevel3.js`, let's add our Game Manager by adding the following import at the top:

```js
    import MyGameManager from './MyGameManager';
```

Then, in each of those level scene files, add the following lines in your create method:

```js
    // setup game manager
    this.addEntity('gameManager')
        .addComponent(MyGameManager);
```