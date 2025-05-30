---
title: Building a Basic Scene
section_title: Guides
type: guides
layout: docs
parent_section: docs
order: 1
section_order: 2
nav_expand_sections: true
image:
  src: https://user-images.githubusercontent.com/674727/29746371-b7906dcc-8a8c-11e7-8ef4-3f7e4d224ce7.jpg
examples:
  - title: Basic Guide with Environment
    src: https://github.com/aframevr/aframe/tree/master/examples/docs/basic-scene/index.html
---

[primitives]: ../introduction/html-and-primitives.md
[position]: ../components/position.md
[rotation]: ../components/rotation.md
[scale]: ../components/scale.md

Let's start by building a basic A-Frame scene. For this, we will need a basic
understanding of HTML. We will learn how to:

- Add 3D entities (i.e., objects) with [primitives][primitives]
- Transform entities in 3D space with position, rotation, scale
- Add an environment
- Add textures
- Add basic interactivity using animations and events
- Add text

[Remix the Basic Guide example on Glitch](https://glitch.com/~aframe-basic-guide-with-environment).

<!--toc-->

## Starting with HTML

We start out with a minimal HTML structure:

```html
<html>
  <head>
    <script src="https://aframe.io/releases/1.7.1/aframe.min.js"></script>
  </head>
  <body>
    <a-scene>
    </a-scene>
  </body>
</html>
```

We include A-Frame as a script tag in the `<head>`, pointing to the A-Frame
build hosted on a CDN. This has to be included *before* `<a-scene>` because
A-Frame registers custom HTML elements which must be defined before `<a-scene>`
is attached or else `<a-scene>` will do nothing.

[scene]: ../core/scene.md

Next, we include [`<a-scene>`][scene] in the `<body>`. `<a-scene>` will contain every
entity in our scene. `<a-scene>` handles all of the setup that is required for
3D: setting up WebGL, the canvas, camera, lights, renderer, render loop as well
as out of the box WebXR support on headsets and WebXR enabled browser on smartphones.
`<a-scene>` alone takes a lot of load off of us!

## Adding an Entity

[abox]: ../primitives/a-box.md

Within our `<a-scene>`, we attach 3D entities using one of A-Frame's standard
[primitives] `<a-box>`. We can use `<a-box>` just like a normal HTML element,
defining the tag and using HTML attributes to customize it. Some other examples
of primitives that come with A-Frame include `<a-cylinder>`, `<a-plane>`, or
`<a-sphere>`.

Here we define the color `<a-box>`, see [`<a-box>`'s documentation][abox] for
the more attributes (e.g., `width`, `height`, `depth`).

[boximage]: https://cloud.githubusercontent.com/assets/674727/20326452/b7f94966-ab3d-11e6-95e1-47cabf425278.jpg
![boximage]
<small class="image-caption"><i>Image by Ruben Mueller from vrjump.de</i></small>

```html
<a-scene>
  <a-box color="red"></a-box>
</a-scene>
```

[entity]: ../core/entity.md
[geometry]: ../components/geometry.md
[material]: ../components/material.md

As a side note, primitives are A-Frame's easy-to-use HTML elements that wrap
the underlying entity-component assembly. They can be convenient, but
underneath `<a-box>` is [`<a-entity>`][entity] with the [geometry] and
[material] components:

```html
<a-entity id="box" geometry="primitive: box" material="color: red"></a-entity>
```

[camera]: ../components/camera.md

However, because the default [camera] and the box are positioned at the default
position at the `0 0 0` origin, we won't be able to see the box unless we move
it. We can do this by using the [position][position] component to transform the
box in 3D space.

## Transforming an Entity in 3D

Let's first go over 3D space. A-Frame uses a right-handed coordinate system.
With the default camera direction: positive X-axis extends *right*, positive
Y-axis extends *up*, and the positive Z-axis extends *out* of the screen
towards us:

[righthandimage]: https://cloud.githubusercontent.com/assets/674727/20328731/326137a8-ab49-11e6-9b76-4e3a65f333d9.jpg
![righthandimage]
<small class="image-caption"><i>Image from what-when-how.com</i></small>

[webvr]: https://developer.mozilla.org/docs/Web/API/WebVR_API

A-Frame's distance unit is in **meters** because the [WebVR
API][webvr] returns pose data in meters. When designing a scene for VR, it is
important to consider the real world scale of the entities we create. A box with
`height="10"` may look normal on our computer screens, but in VR the box will
appear massive.

[right-hand rule]: https://wikipedia.org/wiki/Right-hand_rule

A-Frame's rotational unit in A-Frame is in **degrees**, although it will get
internally converted to radians when passing to three.js. To determine the
positive direction of rotation, use the [right-hand rule]. Point our thumbs
down the direction of a positive axis, and the direction which our fingers curl
around the positive direction of rotation.

To translate, rotate, and scale the box, we can change the
[position][position], [rotation][rotation], and [scale][scale] components.
Let's first apply the rotation and scale components:

```html
<a-scene>
  <a-box color="red" rotation="0 45 45" scale="2 2 2"></a-box>
</a-scene>
```

This will rotate our box at an angle and double its size.

### Parent and Child Transforms

[scene graph]: https://wikipedia.org/wiki/Scene_graph

A-Frame HTML represent a 3D [scene graph]. In a scene graph, entities can have
a single parent and multiple children. Child entities inherit transformations
(i.e., position, rotation, and scale) from their parent.

For example, we could have a sphere as a child of a box:

```html
<a-scene>
  <a-box position="0 2 0" rotation="0 45 45" scale="2 4 2">
    <a-sphere position="1 0 3"></a-sphere>
  </a-box>
</a-scene>
```

If we calculate the sphere's world position, it would be `1 2 3`, achieved by
composing the sphere's parent position with its own position. Similarly, for
rotation and scale, the sphere would inherit the box's rotation and scale. The
sphere too would be rotated and stretched just as like its parent box. If the
box were to change its position, rotation, or scale, it would immediately apply
to and affect the sphere.

If we were to add a cylinder as a child to our sphere, the cylinder's transform
would be affected by both the sphere's *and* box's transforms. Under the hood
in three.js, this is done by multiplying *transformation matrices* together.
Fortunately, we don't have to think about that!

### Placing our Box in Front of the Camera

Now let's get back to making our box visible to the camera from the start. We
can move the box back *5 meters* on the negative Z-axis with the position
component. We also have to move it up *2 meters* on the positive Y-axis so the
box doesn't intersect with the ground since we scaled the box and scaling happens
from the center:

```html
<a-scene>
  <a-box color="red" position="0 2 -5" rotation="0 45 45" scale="2 2 2"></a-box>
</a-scene>
```

Now we see our box!

[boximage2]: https://cloud.githubusercontent.com/assets/674727/20327511/34a5e5f6-ab42-11e6-8107-66c42b80f846.png
![boximage2]

### Default Controls

For flat displays (i.e., laptop, desktop), the default control scheme lets us
look around by click-dragging the mouse and move around with the `WASD` or
arrow keys. On mobile, we can pan the phone around to rotate the camera.
Although A-Frame is tailored for WebVR, this default control scheme allows
people to view scenes without a headset.

[webvrimage]: https://cloud.githubusercontent.com/assets/674727/20328949/69e2c628-ab4a-11e6-97e9-6e25104d6673.png
![webvrimage]

Upon entering VR by clicking the goggles icon with a VR headset
connected (e.g., Oculus Rift, HTC Vive), we can experience the scene in
immersive VR. If room-scale is available, we can physically *walk* around the
scene!

## Adding an Environment

[environment]: https://github.com/supermedium/aframe-environment-component/

A-Frame allows developers to create and share reusable components for others to
easily use. The [environment component][environment] procedurally
generates a variety of entire environments for us with a single line of HTML.
The environment component is a great and easy way to visually bootstrap our VR
application, providing over a dozen environments with numerous parameters:

First, include the environment component using a script tag after A-Frame:

```html
<head>
  <script src="https://aframe.io/releases/1.7.1/aframe.min.js"></script>
  <script src="https://unpkg.com/aframe-environment-component@1.5.x/dist/aframe-environment-component.min.js"></script>
</head>
```

Then within the scene, add an entity with the environment component attached.
We can specify a preset (e.g., `forest`) with along many other parameters
(e.g., 200 trees):

```html
<a-scene>
  <a-box color="red" position="0 2 -5" rotation="0 45 45" scale="2 2 2"></a-box>

  <!-- Out of the box environment! -->
  <a-entity environment="preset: forest; dressingAmount: 500"></a-entity>
</a-scene>
```

## Applying an Image Texture

> Make sure you're [serving your HTML using a local server](../introduction/installation.md#use-a-local-server)
> for textures to load properly.
> Due to an [issue with imgur.com](https://stackoverflow.com/questions/43895390/imgur-images-returning-403), view the page using http://localhost, rather than http://127.0.0.1

We can apply an image texture to the box with an image, video, or `<canvas>`
using the `src` attribute, just like we would with a normal `<img>` element.
We also should remove the `color="red"` that we set so that the color doesn't
get blended in with the texture. The default material color is `white`, so
removing the color attribute is good enough.

```html
<a-scene>
  <a-box src="https://i.imgur.com/mYmmbrp.jpg" position="0 2 -5" rotation="0 45 45"
    scale="2 2 2"></a-box>

  <a-sky color="#222"></a-sky>
</a-scene>
```

Now we'll see our box with an image texture pulled from online:

[boxtextureimage]: https://cloud.githubusercontent.com/assets/674727/20329111/368aea3e-ab4b-11e6-89e1-d13c994290ce.png
![boxtextureimage]

### Using the Asset Management System

[asset management system]: ../core/asset-management-system.md

However, we recommend using the [asset management system] for performance. The
asset management system makes it easier for the browser to cache assets (e.g.,
images, videos, models) and A-Frame will make sure all of the assets are
fetched before rendering.

If we define an `<img>` in the asset management system, later three.js doesn't
have to internally create an `<img>`. Creating the `<img>` ourselves also gives
us more control and lets us reuse the texture across multiple entities. A-Frame
is also smart enough to set `crossOrigin` and other such attributes when
necessary.

To use the asset management system for an image texture:

- Add `<a-assets>` to the scene.
- Define the texture as an `<img>` under `<a-assets>`.
- Give the `<img>` an HTML ID (e.g., `id="boxTexture"`).
- Reference the asset using the ID in DOM selector format (`src="#boxTexture"`).

```html
<a-scene>
  <a-assets>
    <img id="boxTexture" src="https://i.imgur.com/mYmmbrp.jpg">
  </a-assets>

  <a-box src="#boxTexture" position="0 2 -5" rotation="0 45 45" scale="2 2 2"></a-box>

  <a-sky color="#222"></a-sky>
</a-scene>
```

## Creating a Custom Environment (Optional)

Previously we had the environment component generate the environment. Though
it's good to know a bit on creating a basic environment for learning purposes.

### Adding a Background to the Scene

[sky]: ../primitives/a-sky.md

We can add a background with [`<a-sky>`][sky] that surrounds the scene.
`<a-sky>`, which is a material applied to the inside of a large sphere, can be
a flat color, a 360&deg; image, or a 360&deg; video. For example, a dark gray
background would be:

```html
<a-scene>
  <a-box color="red" position="0 2 -5" rotation="0 45 45" scale="2 2 2"></a-box>

  <a-sky color="#222"></a-sky>
</a-scene>
```

Or we can use an image texture to get a 360&deg; background image by using
`src` instead of `color`:

```html
<a-scene>
  <a-assets>
    <img id="boxTexture" src="https://i.imgur.com/mYmmbrp.jpg">
    <img id="skyTexture"
      src="https://cdn.aframe.io/360-image-gallery-boilerplate/img/sechelt.jpg">
  </a-assets>

  <a-box src="#boxTexture" position="0 2 -5" rotation="0 45 45" scale="2 2 2"></a-box>

  <a-sky src="#skyTexture"></a-sky>
</a-scene>
```

### Adding a Ground

[aplane]: ../primitives/a-plane.md

To add a ground, we can use [`<a-plane>`][aplane]. By default, planes are
oriented parallel to the XY axis. To make it parallel to the ground, we need to
orient it along the XZ axis. We can do so by rotating the plane negative
90&deg; on the X-axis.

```html
<a-plane rotation="-90 0 0"></a-plane>
```

We'll want the ground to be large, so we can increase the `width` and `height`. Let's
make it 30-meters in each direction:

```html
<a-plane rotation="-90 0 0" width="30" height="30"></a-plane>
```

Then we can apply an image texture to our ground:

```html
  <a-assets>
    <!-- ... -->
    <img id="groundTexture" src="https://cdn.aframe.io/a-painter/images/floor.jpg">
    <!-- ... -->
  </a-assets>

  <!-- ... -->
  <a-plane src="#groundTexture" rotation="-90 0 0" width="30" height="30"></a-plane>
  <!-- ... -->
```

To tile our texture, we can use the `repeat` attribute. `repeat` takes two
numbers, how many times to repeat in the X direction and how many times to
repeat in the Y direction (commonly referred to as U and V for textures).

```html
<a-plane src="#groundTexture" rotation="-90 0 0" width="30" height="30"
  repeat="10 10"></a-plane>
```

### Tweaking Lighting

[light]: ../primitives/a-light.md

We can change how the scene is lit by using [`<a-light>s`][light]. By default
if we don't specify any lights, A-Frame adds an ambient light and a directional
light. If A-Frame didn't add lights for us, the scene would be black. Once we
add lights of our own, however, the default lighting setup is removed and
replaced with our setup.

We'll add an ambient light that has a slight blue-green hue that matches the
sky. Ambient lights are applied to all entities in the scene (given they have
the default material applied at least).

We'll also add a point light. Point lights are like light bulbs; we can
position them around the scene, and the effect of the point light on an entity
depends on its distance to the entity:

```html
<a-scene>
  ...
  <a-light type="ambient" color="#445451"></a-light>
  <a-light type="point" intensity="2" position="2 4 4"></a-light>
  ...
</a-scene>
```

## Adding Animation

[animation]: ../components/animation.md

We can add animations to the box using A-Frame's [built-in animation
system][animation]. Animations interpolate or tween a value over time. We can
set the animation component on the entity. Let's have the box hover up and down
to add some motion to the scene.

```html
<a-scene>
  <a-assets>
    <img id="boxTexture" src="https://i.imgur.com/mYmmbrp.jpg">
  </a-assets>

  <a-box src="#boxTexture" position="0 2 -5" rotation="0 45 45" scale="2 2 2"
         animation="property: object3D.position.y; to: 2.2; dir: alternate; dur: 2000; loop: true"></a-box>
</a-scene>
```

[entity]: ../core/entity.md
[object3D]: ../core/entity.md#object3d

We tell the animation component to:

- Animate the [entity's][entity] [object3D's][object3D] position's Y axis.
- Animate *to* `2.2` which is 20 centimeters higher.
- Alternate the *dir* (direction) of the animation on each repeated cycle of the animation so it goes back and forth.
- Last for 2000 millisecond *dur* (duration) on each cycle.
- *Loop* or repeat the animation forever.

[animationgif]: https://cloud.githubusercontent.com/assets/674727/20375894/e8b3fa48-ac36-11e6-9ecd-5a5fe33cb9db.gif
![animationgif]

## Adding Interaction

[cursor component]: ../components/cursor.md

Let's add interaction with the box: when we look at the box, we'll increase the
size of the box, and when we "click" on the box, we'll make it spin.

Given that many developers currently do not have proper VR hardware with
controllers, we'll focus this section on using basic mobile and desktop inputs
with the built-in [cursor component]. The cursor component by default provides
the ability to "click" on entities by staring or gazing at them on mobile, or
on desktop, looking at an entity and click the mouse. But know that the cursor
component is just one way to add interactions, things open up if we have access
to actual controllers.

[parentchild]: #parent-and-child-transforms

To have a visible cursor fixed to the [camera], we place the cursor as a child
of the camera as explained above in [Parent and Child Transforms][parentchild].

[camera]: ../components/camera.md

Since we didn't specifically define a camera, A-Frame included a default camera
for us. But since we need to add a cursor as a child of the camera, we will
need to now define [`<a-camera>`][camera] containing `<a-cursor>`:

```html
<a-scene>
  <a-assets>
    <img id="boxTexture" src="https://i.imgur.com/mYmmbrp.jpg">
  </a-assets>

  <a-box src="#boxTexture" position="0 2 -5" rotation="0 45 45" scale="2 2 2"
         animation="property: object3D.position.y; to: 2.2; dir: alternate; dur: 2000; loop: true"></a-box>

  <a-camera>
    <a-cursor color="#FAFAFA"></a-cursor>
  </a-camera>
</a-scene>
```

If we check the [documentation of the cursor component][cursor component] that
`<a-cursor>` wraps, we see that it emits hover events such as `mouseenter`,
`mouseleave` as well as `click`.

### Event Listener Component (Intermediate)

[addeventlistener]: https://developer.mozilla.org/docs/Web/API/EventTarget/addEventListener
[animatingonevents]: #animating-on-events
[queryselector]: https://developer.mozilla.org/docs/Web/API/Document/querySelector
[setattribute]: ../core/entity.md#setattribute-componentname-value-propertyvalue-clobber

One way to manually handle the cursor events is to [add an event listener with
JavaScript][addeventlistener] just like we would with a normal DOM element. If
you aren't comfortable with JavaScript, you may skip to [Animating on
Events][animatingonevents] below.

In JavaScript, we grab the box with [`querySelector`][queryselector], use
[`addEventListener`][addeventlistener], and then [`setAttribute`][setattribute]
to make the box grow its scale when its hovered over. Note that A-Frame adds
features to `setAttribute` to work with multi-property components. We can pass
a full `{x, y, z}` object as the second argument.

```js
<script>
  var boxEl = document.querySelector('a-box');
  boxEl.addEventListener('mouseenter', function () {
    boxEl.setAttribute('scale', {x: 2, y: 2, z: 2});
  });
</script>
```

But a much more robust method would be to encapsulate this logic into an
A-Frame component. This way, we don't have to wait for the scene to load, we
don't have to run query selectors because components give us context, and
components can be reused and configured versus having an uncontrolled script
running on the page. And better yet would be to skip calling `.setAttribute`
and set the value on the `this.el.object3D.scale` directly for performance.

```js
<script>
  AFRAME.registerComponent('scale-on-mouseenter', {
    schema: {
      to: {default: '2.5 2.5 2.5', type: 'vec3'}
    },

    init: function () {
      var data = this.data;
      var el = this.el;
      this.el.addEventListener('mouseenter', function () {
        el.object3D.scale.copy(data.to);
      });
    }
  });
</script>
```

We can attach this component to our box straight from HTML:

```html
<script>
  AFRAME.registerComponent('scale-on-mouseenter', {
    // ...
  });
</script>

<a-scene>
  <!-- ... -->
  <a-box src="#boxTexture" position="0 2 -5" rotation="0 45 45" scale="2 2 2"
         animation="property: object3D.position.y; to: 2.2; dir: alternate; dur: 2000; loop: true"
         scale-on-mouseenter></a-box>
  <!-- ... -->
</a-scene>
```

### Animating on Events

The animation component has a feature to start its animation when the entity
emits an event. This can be done through the `startEvents` attribute, which
takes a comma-separated list of event names.

We can add two animations for the cursor component's `mouseenter` and 
`mouseleave` events to change the box's scale, and one for rotating the box
around the Y-axis on `click`. Note that an entity can have multiple animations
by suffixing the attribute name with `__<ID>`:

```html
<a-box
  src="#boxTexture"
  position="0 2 -5"
  rotation="0 45 45"
  scale="2 2 2"
  animation__position="property: object3D.position.y; to: 2.2; dir: alternate; dur: 2000; loop: true"
  animation__mouseenter="property: scale; to: 2.3 2.3 2.3; dur: 300; startEvents: mouseenter"
  animation__mouseleave="property: scale; to: 2 2 2; dur: 300; startEvents: mouseleave"
  animation__click="property: rotation; from: 0 45 45; to: 0 405 45; dur: 1000; startEvents: click"></a-box>
```

## Adding Audio

Audio is important for providing immersion and presence in VR. Even adding
simple white noise in the background goes a long way. We recommend having some
audio for every scene. One way would be to add an `<audio>` element to our
HTML (preferably under `<a-assets>`) to play an audio file:

```html
<a-scene>
  <a-assets>
    <audio src="https://cdn.aframe.io/basic-guide/audio/backgroundnoise.wav" autoplay
      preload></audio>
  </a-assets>

  <!-- ... -->
</a-scene>
```

Or we can add positional audio using `<a-sound>`. This makes the sound get
louder as we approach it and get softer as we distance from it. We could place
the sound in our scene using `position`.

```html
<a-scene>
  <!-- ... -->
  <a-sound src="https://cdn.aframe.io/basic-guide/audio/backgroundnoise.wav" autoplay="true"
    position="-3 1 -4"></a-sound>
  <!-- ... -->
</a-scene>
```

## Adding Text

[text]: ../components/text.md
[three-bmfont-text]: https://github.com/Jam3/three-bmfont-text

A-Frame comes with a [text component][text]. There are several ways to render
text, each with their own advantages and disadvantages. A-Frame comes with an SDF
text implementation using [`three-bmfont-text`][three-bmfont-text] that is
relatively sharp and performant:

```html
<a-entity text="value: Hello, A-Frame; color: #FAFAFA; width: 5; anchor: align"
          position="-0.9 0.2 -3"
          scale="1.5 1.5 1.5"></a-entity>
```

## Play With It!

And that's the basic example!

[View the example](https://aframe.io/aframe/examples/docs/basic-scene/) (basic environment)

[View the example](https://aframe.io/aframe/examples/docs/basic-scene-2/) (custom environment)
