# Animation

The `Animation` component manages sprite animation clips. It stores a `Map` of named clips, each with an array of frames (images), FPS, and loop setting. The `AnimationSystem` advances the current clip each frame and updates `entity.renderable.image`.

## Constructor

```js
const anim = new Animation()
```

## Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `animations` | `Map` | empty | Named clip definitions |
| `current` | `string \| null` | `null` | Name of the currently playing clip |
| `frame` | `number` | `0` | Current frame index within the clip |
| `elapsed` | `number` | `0` | Accumulated time since last frame advance |
| `playing` | `boolean` | `false` | Whether the animation is actively advancing |

## Methods

### `add(name, clip)`

```js
anim.add(name, clip)  // returns this
```

Adds a named animation clip. The `clip` object:

| Clip Property | Type | Description |
|---------------|------|-------------|
| `frames` | `HTMLImageElement[]` | Array of images, one per frame |
| `fps` | `number` | Frames per second play rate |
| `loop` | `boolean` | Whether to loop on completion |

```js
anim.add('walk', {
  frames: [walk1, walk2, walk3, walk4],
  fps: 8,
  loop: true,
})
```

### `play(name)`

```js
anim.play('walk')
```

Start playing a named clip. Resets `frame` and `elapsed` to `0`.

### `pause()`

```js
anim.pause()
```

Pauses frame advancement without resetting state.

### `resume()`

```js
anim.resume()
```

Resumes playback of the current clip (no-op if no clip is set).

### `stop()`

```js
anim.stop()
```

Stops playback and resets `frame` and `elapsed` to `0`.

### `onComplete(cb)`

```js
anim.onComplete(() => console.log('Animation finished'))  // returns this
```

Registers a one-time callback fired when a non-looping clip reaches its final frame. The callback is automatically cleared after firing.

## Usage

```js
import { Sprite, AnimationSystem, animationSystem } from 'jygame'

const player = new Sprite(100, 100, 32, 48)

// Add animation clips
player.animation.add('idle', {
  frames: [idleFrame],
  fps: 1,
  loop: true,
})
player.animation.add('run', {
  frames: [run1, run2, run3, run4],
  fps: 10,
  loop: true,
})

player.animation.play('idle')

// In scene update — AnimationSystem advances frames automatically
scene.update = function (dt) {
  animationSystem.updateOne(player, dt)
  // player.renderable.image is updated to the current frame
}
```
