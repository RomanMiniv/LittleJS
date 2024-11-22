# LittleJS FAQ

Welcome to the LittleJS FAQ!
This document addresses common questions and issues to help developers get started and troubleshoot their projects.
If you don’t find an answer here, feel free to ask the community or check the documentation.

---

## Getting Started
### What is LittleJS, and how is it different from other JavaScript game engines?

LittleJS is a lightweight and fast JavaScript game engine designed for simplicity and super fast sprite rendering.
It focuses on 2D games and provides essential features like game objects, particle effects, and physics out of the box.
Unlike larger engines, LittleJS has a small footprint and avoids unnecessary complexity, making it perfect for quick prototyping or smaller projects.
However it is also very fast and extendable making it a great choice even for larger games!

### How do I set up a basic LittleJS project?

Download the LittleJS repository via GitHub or npm.
Include one of the LittleJS builds from the dist folder.
Several examples are included for you to build on.
The most basic example is just an empty project.

[Empty Example HTML file:](https://github.com/KilledByAPixel/LittleJS/blob/main/examples/empty/index.html)
```html
<!DOCTYPE html><head>
<title>LittleJS Hello World Demo</title>
<meta charset=utf-8>
</head><body>

<script src=../../dist/littlejs.js></script>
<script src=game.js></script>
```

[Empty Example JavaScript file:](https://github.com/KilledByAPixel/LittleJS/blob/main/examples/empty/game.js)
```javascript
/*
    Little JS Hello World Demo
    - Just prints "Hello World!"
    - A good starting point for new projects
*/

'use strict';

///////////////////////////////////////////////////////////////////////////////
function gameInit()
{
    // called once after the engine starts up
    // setup the game
}

///////////////////////////////////////////////////////////////////////////////
function gameUpdate()
{
    // called every frame at 60 frames per second
    // handle input and update the game state
}

///////////////////////////////////////////////////////////////////////////////
function gameUpdatePost()
{
    // called after physics and objects are updated
    // setup camera and prepare for render
}

///////////////////////////////////////////////////////////////////////////////
function gameRender()
{
    // called before objects are rendered
    // draw any background effects that appear behind objects
}

///////////////////////////////////////////////////////////////////////////////
function gameRenderPost()
{
    // called after objects are rendered
    // draw effects or hud that appear above all objects
    drawTextScreen('Hello World!', mainCanvasSize.scale(.5), 80);
}

///////////////////////////////////////////////////////////////////////////////
// Startup LittleJS Engine
engineInit(gameInit, gameUpdate, gameUpdatePost, gameRender, gameRenderPost, ['tiles.png']);
```

### Why do I see a blank screen when I run my game?

If you are seeing a blank screen, first try opening the dev tools console (F12 in most browsers).
This will show you any errors that occur and allows stepping through code to help debug.
A common issue is the image data failing to load with a message like "The image element contains cross-origin data, and may not be loaded."
This is probably because the game was loaded directly without using a web server.

### Do I need a local server to run LittleJS games, and how do I set one up?

Yes, this is a necessary step because web browsers just have protection from loading local files which includes images.
So any JavaScript projects that load images like games must be opened from a local web server.
Don't panic though, it's very easy to fix! 

If you are using [Visual Studio Code](https://code.visualstudio.com/) there is a [Live Preview Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.live-server) that will handle this for you automatically.

Another option is to setup a simple local web server like [http-server](https://www.npmjs.com/package/http-server) via npm.

---

## Graphics and Sound

### How do I load and add images to my game?

First you need to load an image file. For LittleJS this is typically done on startup via a parameter to engineInt that is a list of images to load. The engine will ensure that the images are all loaded before starting. Each source image can be up to 4096x4096 in size so most games only need one texture, though its possible to load as many as you need.

```javascript
engineInit(gameInit, gameUpdate, gameUpdatePost, gameRender, gameRenderPost, ['tiles.png']);
```

LittleJS works best when your tile sheet is broken up into grids of tiles because the rendering system can be batched up. To draw a tile from a source image you can call drawTile and pass in TileInfo object. Another common approach is to create an EngineObject and set it's tileInfo, it will automatically be rendered.

```javascript
drawTile(vec2(21,5), vec2(4.5), tile(3,128));
```

### What is the tile function and how do tile indexes work?

The tile function is a very useful function that takes a tile index and size and returns a TileInfo object which can be passed to functions like drawTile. It works by accepting using the tileIndex multiplied by the tileSize to get the coordinates for the TileInfo. It's also possible to pass in padding for sheets that are set up for it.

### Can I add and switch between multiple sprites for a game object?

You can set the object's tileInfo to a new sprite, or just call drawTile directly with any sprite.

```javascript 
this.tileInfo = tile(3, 32);
```

### How do I handle animations in LittleJS?

You can use the TileInfo.frame function passing in the number of animation frames to offset the sprite.
It sounds kind of weird at first but imagine your sprites are all aligned on a grid with frames of animations being next to eachother.
This allows easily indexing into those animations from the base sprite location.
For example to animate the player sprite you might do something like this...

```javascript 
this.tileInfo = spriteAtlas.player.frame(animationFrame);
```

### How do I control the camera in LittleJS?

LittleJS uses a world space rendering system, so objects can move independently of the camera.
The camera is easy to control using cameraPosition and cameraScale, which indicate the world space position and how many pixels is equivalent to one world unit.
It is also possible to draw using screenSpace pixel coordinates by passing in true as the screenSpace parameter to most drawing functions.

### How do I play sounds in LittleJS?

Sounds can either be generated on startup using ZzFX or loaded from a wave or mp3 file.
[ZzFX sounds can be created using the sound designer app.](https://killedbyapixel.github.io/ZzFX/)
Once loaded sounds can be played by calling Sound.play with some paratermers to control how it is played.

```javascript
const sound_click = new Sound([1,.5]); // create a ZzFX sound
const sound_jump = new SoundWave('jump.mp3'); // load an mp3 sound
sound_click.play(pos, volume, pitch); // play a sound
```

### If I load several images, how do I control which is used?

There is just an optional parameter you can pass to the tile function or to TileInfo called textureIndex. It defaults to 0 so if you only have one texture, you may not have noticed it. If you set to another number will select another texture in list. Something to keep in mind for large games is that changing the target texture will require WebGL to flush the render batch which can cause slowdown if it is done many times per frame.

```javascript
function tile(pos=vec2(), size=tileSizeDefault, textureIndex=0, padding=0)
```

### How can I check if an object is on screen?

You can use the isOverlapping function to check the object against the camera. For culling you maybe want to enlarge the object size slightly to account for attached objects or rotation, I usually do this.size.scale(2).

```javascript
if (!isOverlapping(this.pos, this.size, cameraPos, renderWindowSize))
    return;
```

---

## Gameplay and Programming

### How do I add keyboard or mouse input to my game?

LittleJS provides input handling functions for keyboard, mouse, and gamepads. Touch input is also routed to the mouse. There are functions for isDown, wasPressed, and wasReleased.

```javascript
if (keyIsDown(37)) // Left arrow key
   obj.x -= 5;
if (mouseWasReleased(0)) // Left mouse button
   console.log('Mouse clicked at:', mousePos);
if (gamepadWasPressed(0)) // Gamepad button 0
   console.log('Gamepad pressed, stick is:', gamepadStick(0));
```

### How do I create and update game objects?

You can create custom objects for your game by extending the EngineObject class. They will automatically be updated and rendered by the engine

```javascript
class MyObject extends EngineObject 
{
    constructor(pos, size, tileInfo)
    {
        super(pos, size, tileInfo);
        // setup object
    }

    update()
    {
        // update object physics and position
        super.update(); 
    }

    render()
    {
        // draw object as a sprite
        super.render();
    }
}

// spawn one of those objects
const object = new MyObject(spawnPos, size, tile);
```

### Can I use physics with LittleJS?

Yes! LittleJS comes with a robust game physics system included and [also a plugin using Box2D.](https://killedbyapixel.github.io/LittleJS/examples/box2d/)
The platformer example includes a character object class that can be used as a starting point for advanced platforming physics.

### How do I add particle effects to my game?

[There is a particle system designer that is useful for experimenting with particle designs.](https://killedbyapixel.github.io/LittleJS/examples/particles/)

You can  create a particle system in code using the ParticleEmitter object.

```javascript
// fire particle system
new ParticleEmitter(
    pos, 0,                         // pos, angle
    1, .1, 100, PI,                 // emitSize, emitTime, emitRate, emiteCone
    0,                              // tileInfo
    rgb(1,.5,.1), rgb(1,.1,.1),     // colorStartA, colorStartB
    rgb(1,.5,.1,0), rgb(1,.1,.1,0), // colorEndA, colorEndB
    .7, .8, .2, .2, .05,  // time, sizeStart, sizeEnd, speed, angleSpeed
    .9, 1, -.2, PI, .05,  // damp, angleDamp, gravity, particleCone, fade
    .5, 0, 1, 0, 1e9      // randomness, collide, additive, colorLinear, renderOrder
);
```

---

## Debugging and Development

### How do I debug my game in LittleJS?

Press the Esc key to show the the debug menu. From here there are several options that can be accesed via the number keys.

You can also press + or - to speed up or slow down time to help with debugging!

---

## Contribute to the FAQ
If you have additional questions or think something should be added to this FAQ, please open an issue or pull request on the [LittleJS GitHub repository](https://github.com/FrankForce/LittleJS).

---

Happy coding with LittleJS! 🎮🚂💨