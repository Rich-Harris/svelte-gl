# svelte-gl

This is not a thing. I would like it to become a thing one day, if time allows, but for now I just want to get some ideas out of my head and into a place where they can be examined.

The idea is simple: a cross between [Svelte](https://svelte.technology/) and [Three.js](https://threejs.org/). In other words, a compiler that takes a scene graph (written in [HTMLx](https://github.com/htmlx-org/HTMLx)) as input, and generates low-level WebGL code as output.

[A-Frame](https://aframe.io/) is strong evidence that markup-driven approaches to describing 3D scenes are intuitive for authors, and practical from a performance standpoint. But A-Frame and Three.js, while truly excellent, are large frameworks (in terms of JavaScript bundle size) that are therefore unsuitable for *casual* or *whimsical* uses of WebGL.

My hypothesis is that we can make it much easier to embed 3D graphics on the web in a way that is potentially *more* performant than existing solutions (because compilers) at a fraction of the cost in terms of bytes, and insodoing unlock new forms of creativity.

It's entirely possible that I am completely wrong about this.


## API ideas

```html
<!-- every scene must have a camera, but one could
     be created automatically at a sensible location
     if not specified, a la A-Frame -->
<camera
  position={cameraPos}
  target="#my-box"
/>

<!-- position defaults to { x: 0, y: 0, z: 0 } -->
<box
  id="my-box"
  size={2}
  on:click="set({ color: 'blue' })"
  {color}
/>

<pointlight
  position="{{ x: 10, y: 10, x: 10 }}"
  target="#my-box"
/>
```

```js
import Scene from './Scene.htmlx';

const scene = new Scene({
  canvas: document.querySelector('canvas'),
  data: {
    cameraPos: { x: 0, y: 0, z: -5 },
    color: red
  }
});

// later — trigger a re-render
scene.set({
  cameraPos: { x: -2, y: 4, z: -3 }
});
```

We could adopt large chunks of the Svelte component API, for handling local state, lifecycle hooks and the like.

## Interop with Svelte components

It would be pretty great if we could do this sort of thing:

```html
<h1>this is a svelte-gl component</h1>

<canvas>
  <MyScene {color}/>
</canvas>

<input type=color bind:color/>

<script>
   export default {
      components: {
         MyScene: './MyScene.gl.htmlx'
      }
   }
</script>
```


## Implementation

Err, TODO. A couple of high level thoughts though.

Firstly, the job is easier in some respects than Svelte's DOM manipulation — rather than surgically updating parts of the document, in WebGL land we just re-render the entire world on any state change.

But the flip side is that it's probably not enough to just convert each element in the graph into a draw call; we probably need to do more work up-front to figure out there are no constraint violations (e.g. in the example above, we need to ensure that `#my-box` refers to an element that currently exists, and figure out its position so that it can be set as the target of the camera and the light. Though maybe it should be `target={boxPos}` anyway).

By far the easiest way to get started — and probably the most sensible, long-term — would be to add a new compiler target within Svelte itself. That way we wouldn't need to reimplement a bunch of stuff, and the two projects would stay in sync. The danger is that the experiment doesn't pan out, and Svelte is left with a vestigial appendage, so it would require buy-in from the Svelte community. The alternative would be to extract the different compiler targets out from Svelte into separate packages (`@sveltejs/compile-dom`, `@sveltejs/compile-ssr`, `@sveltejs/compile-gl`) that the core calls out to. I haven't looked into how feasible that would be.

## Misc thoughts

* VoodooJS — an example of mixing 2D with 3D content. No idea what became of this. Would be cool to be able to do the same thing. https://codepen.io/cysidus/pen/hoCzb
* Post-processing?
* WebVR is obviously a must-have... that also means `requestAnimationFrame` won't cut the mustard for animation helpers
* What does asset loading look like in this scenario?
* How can we bind to events etc? What do they look like?
* How can we make things interactive? How can we get WebGL coords (e.g. some vertex, to which we want to add a DOM tooltip) into screen coords?
* WebGL 1 or 2? Or is that a compiler target?
* Would be cool to use OffscreenCanvas where possible
* Physics. In the same way that Svelte only includes the bits of the framework you're using, svelte-gl could have dependencies on things like http://nphysics.org/ that are only imported if the scene has physics. Incorporating physics would make top-down state updates... interesting
* This might be taking things too far, but... if we're a compiler, then maybe we can do stuff like precompute global illumination maps? That way we wouldn't need things like SSAO post-processors (for static scenes at least), and the results would be higher quality. It'd be extremely difficult, but difficult is better than impossible (which is the case for all existing WebGL frameworks, AFAIK)
* Prior art https://github.com/drcmda/react-three-fiber
