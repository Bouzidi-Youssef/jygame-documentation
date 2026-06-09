# Sprite

The `Sprite` class is a data entity composed of **components** — `Transform`, `Collider`, `Renderable`, `Animation`, and a `velocity` Vec2. Sprites have no built-in `update` or `render` methods — **systems** handle all behavior.

## Constructor

```js
const sprite = new Sprite(x, y, width, height)
```

Creates a sprite with top-left corner at `(x, y)` and the given size. Internally, `transform` is centered at `(x + w/2, y + h/2)`.

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `x` | `number` | Top-left X (getter/setter, converts from transform center) |
| `y` | `number` | Top-left Y (getter/setter, converts from transform center) |
| `width` | `number` | Width (delegates to `collider.width`) |
| `height` | `number` | Height (delegates to `collider.height`) |
| `transform` | `Transform` | Spatial state: `x`, `y`, `rotation`, `scale` |
| `collider` | `Collider` | AABB collision volume: `width`, `height` |
| `renderable` | `Renderable` | Visual state: `image`, `style` |
| `animation` | `Animation` | Animation component: named clips, current frame, playing state |
| `velocity` | `Vec2` | Velocity vector (consumed by `MovementSystem`) |
| `visible` | `boolean` | `true` — set to `false` to skip rendering and collision |
| `angle` | `number` | Rotation in radians (delegates to `transform.rotation`) |
| `scale` | `Vec2` | Scale factor (delegates to `transform.scale`) |
| `image` | `HTMLImageElement \| null` | Optional image (delegates to `renderable.image`) |
| `style` | `object` | `{ fill, shape }` style config (delegates to `renderable.style`) |
| `groups` | `Group[]` | Groups this sprite belongs to |

## Methods

### `kill()`

```js
sprite.kill()
```

Removes the sprite from all groups it belongs to and clears `this.groups`.

## Systems Operating on Sprites

Since `Sprite` is a data entity, you use systems to process it:

```js
import { movementSystem, renderSystem, animationSystem } from 'jygame'

// Each frame
movementSystem.updateOne(player, dt)      // applies velocity to transform
animationSystem.updateOne(player, dt)     // advances animation frames
renderSystem.renderOne(ctx, player)       // draws the sprite
```

## Style

```js
sprite.style.fill = '#B0DE8E'
sprite.style.shape = 'circle'   // 'rect' | 'circle' | 'ellipse'
```

## Usage with Images

```js
const sprite = new Sprite(0, 0, 64, 64)
sprite.image = await ImageLoader.load('/assets/player.png')
```

## Example

```js
import { Game, Scene, Sprite, movementSystem, renderSystem, Input } from 'jygame'

const player = new Sprite(100, 200, 32, 48)
player.style.fill = '#63B44E'

const scene = new Scene()
scene.update = function (dt) {
  if (Input.isDown('RIGHT')) player.velocity.x = 200
  else if (Input.isDown('LEFT')) player.velocity.x = -200
  else player.velocity.x = 0

  movementSystem.updateOne(player, dt)
}

scene.render = function (ctx) {
  ctx.clearRect(0, 0, 800, 600)
  renderSystem.renderOne(ctx, player)
}

const game = new Game({ width: 800, height: 600 })
game.run(scene)
```
