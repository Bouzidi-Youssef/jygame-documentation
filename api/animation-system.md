# AnimationSystem

The `AnimationSystem` advances animation clips for entities that have an `animation` component. It updates `entity.renderable.image` to the current frame and fires completion callbacks on non-looping clips.

A singleton `animationSystem` instance is exported for convenience.

## Constructor

```js
new AnimationSystem()
```

## Methods

### `update(entities, dt)`

```js
animationSystem.update(entities, dt)
```

Iterates over an array of entities and calls `updateOne` on each.

### `updateOne(entity, dt)`

```js
animationSystem.updateOne(entity, dt)
```

Advances one entity's animation. Steps:
1. Skips if `entity.animation` is missing or not playing
2. Reads the current clip's `fps` to compute frame duration (`1 / fps`)
3. Accumulates `dt` into `anim.elapsed`
4. Advances frames while `elapsed >= frameTime`
5. On clip completion: if `loop` is true, resets to frame 0; otherwise stops and fires `onComplete` callback
6. Writes the current frame image to `entity.renderable.image`

## Default Singleton

```js
import { animationSystem } from 'jygame'

// Single entity
animationSystem.updateOne(player, dt)

// Batch
animationSystem.update(allEntities, dt)
```

## Usage

```js
import { AnimationSystem, animationSystem, Sprite } from 'jygame'

const player = new Sprite(100, 100, 32, 48)
player.animation.add('walk', {
  frames: [img1, img2, img3, img4],
  fps: 8,
  loop: true,
})
player.animation.play('walk')

// Each frame, advance the animation
scene.update = function (dt) {
  animationSystem.updateOne(player, dt)
  movementSystem.updateOne(player, dt)
}
```
