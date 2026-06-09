# Core Concepts

## Architecture

Jygame uses a pure **Entity-Component-System (ECS)** model. Entities (like `Sprite`) are data containers composed of components. Dedicated **systems** read and write component data — entities have no behavior of their own.

```
Sprite (data entity)
├── transform: Transform    →  position (x, y), rotation, scale
├── collider: Collider      →  width, height
├── renderable: Renderable  →  image, style
├── animation: Animation    →  clips, current frame, playing state
└── velocity: Vec2          →  movement per second
```

Systems that process entities:

| System | Operates On | Responsibility |
|--------|-------------|----------------|
| `MovementSystem` | entities with `velocity` + `transform` | Applies velocity * dt to position |
| `AnimationSystem` | entities with `animation` + `renderable` | Advances frames, writes `renderable.image` |
| `RenderSystem` | entities with `renderable` + `transform` + `collider` | Applies camera, culls, draws to canvas |
| `CollisionSystem` | entities with `collider` + `transform` + `visible` | Spatial hash management, collision queries |

Any object with the right component shape works with these systems — no base class required.

## Game Loop

Jygame uses a **fixed timestep** game loop driven by `requestAnimationFrame`.

```js
import { Game, Scene } from 'jygame'

const game = new Game({ width: 800, height: 600, fps: 60 })

const scene = new Scene()
scene.update = function (dt) {
  // dt is always 1/60 — deterministic updates
}
scene.render = function (ctx) {
  // Draw everything here
}

game.run(scene)
```

The loop:
1. `requestAnimationFrame` fires — real time is measured
2. Real delta is fed into an internal `Clock` accumulator
3. For each accumulated fixed step, `scene.update(fixedDt)` is called
4. After all updates, `scene.interpolate(alpha)` for smooth rendering
5. `scene.render(ctx)` draws to the canvas
6. Input state is reset

## Scenes & Scene Stack

Scenes organize your game into distinct states (menu, gameplay, pause, game over). They live on a **stack** managed by `Game`.

### Lifecycle Hooks

| Hook | Purpose |
|------|---------|
| `enter()` | Setup — called once when mounted |
| `exit()` | Teardown — called once when unmounted |
| `pause()` | Called when a scene is pushed on top |
| `resume()` | Called when the scene above is popped |
| `update(dt)` | Simulation each fixed timestep |
| `interpolate(alpha)` | Smooth rendering between timesteps |
| `render(ctx)` | Drawing each frame |
| `renderUI()` | Returns HTML string for DOM overlay |

### Stack Operations

Scenes are **single-use** — create a new instance each time.

```js
// Push a pause overlay on top
this.pushScene(new PauseScene())

// Pop back to the previous scene
this.popScene()

// Replace the current scene
this.replaceScene(new GameOverScene())

// Reset the entire stack
this.switchScene(new MenuScene())
```

### Blocking

By default:
- `blocksUpdateBelow = true` — scenes below are paused
- `blocksRenderBelow = false` — all scenes render (bottom to top)

```js
class PauseScene extends Scene {
  constructor() {
    super()
    this.blocksUpdateBelow = true
    this.blocksRenderBelow = false  // game renders underneath
  }
}
```

### Event Cleanup

Use `this.on()` for auto-cleaned event listeners:

```js
this.on(document, 'click', onClick)
this.onSwipe(dir => this.move(dir))
this.onTap(({ x, y }) => this.place(x, y))
```

### DOM UI Layer

```js
scene.renderUI = function () {
  return '<h1 id="score">Score: 0</h1>'
}
```

Use `game.refreshUI()` or `game.patchUI({ score: 'Score: 42' })`.

## Sprites

Sprites are data entities — they have **no `update` or `render` methods**. Systems handle all behavior.

```js
const player = new Sprite(100, 200, 32, 48)
player.style.fill = '#B0DE8E'
player.velocity.set(200, 0)

// Each frame:
movementSystem.updateOne(player, dt)
renderSystem.renderOne(ctx, player)
```

Properties: `x`, `y`, `width`, `height`, `angle`, `scale`, `visible`, `style`, `image`, `transform`, `collider`, `renderable`, `animation`, `velocity`.

## Groups

`Group` is a pure iterable container — no `update` or `render`. Use `for...of` with systems.

```js
const enemies = new Group()
enemies.add(enemy1)
enemies.add(enemy2)

for (const sprite of enemies) {
  movementSystem.updateOne(sprite, dt)
  renderSystem.renderOne(ctx, sprite)
}
```

### Spatial Hash

```js
enemies.useSpatialHash(64)
```

### Collision Queries

```js
const hits = enemies.collideRect(rect)
enemies.collideGroup(walls, (a, b) => a.kill())  // callback zero-alloc
enemies.collideSprite(player).forEach(s => s.kill())
```

## Camera

The `Camera` defines the visible world region. The first camera created becomes `Camera.main` automatically — used by `RenderSystem`.

```js
import { Camera, renderSystem } from 'jygame'

const camera = new Camera(400, 300, 800, 600)
camera.zoom = 2

// Track a player
camera.follow(player)

// Screen/world coordinate conversion
const world = {}
camera.screenToWorld(mouseX, mouseY, world)

// RenderSystem uses Camera.main automatically
renderSystem.render(ctx, allEntities)
```

## Animation

Add named animation clips to any entity with an `animation` component.

```js
const player = new Sprite(100, 100, 32, 48)
player.animation.add('walk', {
  frames: [walk1, walk2, walk3, walk4],
  fps: 8,
  loop: true,
})
player.animation.add('idle', {
  frames: [idleFrame],
  fps: 1,
  loop: true,
})

player.animation.play('idle')

// Each frame
animationSystem.updateOne(player, dt)
// player.renderable.image is updated automatically
```

## Systems

### MovementSystem

```js
movementSystem.updateOne(entity, dt)
movementSystem.update(entities, dt)
```

### AnimationSystem

```js
animationSystem.updateOne(entity, dt)
animationSystem.update(entities, dt)
```

### RenderSystem

Accepts a `Camera` (falls back to `Camera.main`):

```js
renderSystem.render(ctx, entities, camera)
renderSystem.renderOne(ctx, entity, camera)
```

### CollisionSystem

Centralized collision with spatial hash management:

```js
collisionSystem.useSpatialHash(myGroup, myGroup)
collisionSystem.beginFrame()
collisionSystem.collideRect(entities, rect)
collisionSystem.collideGroup(a, b, (x, y) => { /* callback */ })
```

## Input

### Action Bindings

Bind multiple keys to a logical action:

```js
Input.bind('JUMP', 'SPACE')
Input.bind('JUMP', 'W')
Input.bind('JUMP', 'UP')

// Check the action, not the raw key
if (Input.justPressed('JUMP')) player.jump()
```

### Key Queries

```js
Input.isDown('RIGHT')       // held this frame
Input.justPressed('SPACE')  // pressed this frame
Input.justReleased('ENTER') // released this frame
```

### Pointer

```js
Input.x, Input.y            // latest pointer position
Input.isPointerDown         // boolean
Input.pointerCount          // multi-touch
Input.getPointers()         // iterator over active pointers
```

### Gestures

```js
Input.onSwipe(dir => { /* UP/DOWN/LEFT/RIGHT */ })
Input.onTap(({ x, y }) => { /* client coords */ })
```

## Object Pooling

### Pool

Simple object reuse with optional warmup:

```js
const bulletPool = new Pool({
  create: () => new Sprite(0, 0, 8, 8),
  initialSize: 50,
})
const b = bulletPool.acquire()
bulletPool.release(b)
```

### ActivePool

Advanced pool with active tracking, batch ops, and iteration:

```js
const pool = new ActivePool({
  create: () => new Sprite(0, 0, 8, 8),
  initialSize: 50,
})

pool.acquireMany(10, null, (b, i) => { b.x = i * 20 })

// Iterate only active objects
pool.forEachActive(b => renderSystem.renderOne(ctx, b))
pool.updateActive(b => { b.update(dt); if (done(b)) pool.release(b) })
pool.clearActive()
```

## Math Utilities

### Vec2

```js
const v = new Vec2(3, 4)
v.add(new Vec2(1, 2)).scale(2)
v.magnitude()        // ≈ 14.42
v.normalize()
v.rotate(Math.PI / 2)
v.setFrom(other)     // no-alloc copy

Vec2.fromAngle(Math.PI, 50)
Vec2.lerpInto(out, a, b, 0.5)  // pool-friendly
```

### Rect

```js
const r = new Rect(10, 20, 100, 80)
r.collides(other)
r.contains({ x: 50, y: 50 })
r.getCenter(out)    // pool-friendly
```

## Asset Loading

### ImageLoader

```js
const task = ImageLoader.loadAll({ player: 'player.png', enemy: 'enemy.png' })
task.onProgress((loaded, total) => console.log(`${loaded}/${total}`))
const assets = await task
```

### FontLoader

```js
const task = FontLoader.loadAll({ Pixel: '/fonts/pixel.woff2' })
await task
```

### LoadingTask

```js
task.onProgress((loaded, total) => { /* progress bar */ })
const result = await task
```

## Color System

458 named colors organized by family:

```js
Color.CyberYellow         // '#ffd400'
Colors.Green              // '#d6fb61'
Colors.BlueShades.FrostedBlueberries
```
