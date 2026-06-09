# Best Practices

## Project Structure

```
my-game/
├── index.html
├── src/
│   ├── main.js              # Entry point — creates Game, runs initial scene
│   ├── scenes/              # One file per scene
│   │   ├── MenuScene.js
│   │   ├── GameScene.js
│   │   └── PauseScene.js
│   ├── entities/            # Sprite factories or custom entity definitions
│   ├── systems/             # Game logic separated from scenes
│   └── assets/
├── package.json
└── vite.config.js
```

Keep scenes focused. If a scene exceeds 200 lines, extract logic into separate files.

## Scene Lifecycle

### Scenes Are Single-Use

Create a new scene instance each time you enter a state. Never re-use an exited scene.

```js
// ✅ Correct
game.switchScene(new MenuScene())

// ❌ Error: scene already exited
game.switchScene(menuScene)
game.switchScene(menuScene)
```

### Setup in `enter()`, Teardown in `exit()`

```js
class GameScene extends Scene {
  enter() {
    this.player = new Sprite(100, 100, 32, 32)
    this.on(Input, 'keydown', this.handleInput)
  }
}
```

### Use the Scene Stack

Push overlays (pause, inventory) on top instead of switching scenes:

```js
class GameScene extends Scene {
  update(dt) {
    if (Input.justPressed('ESCAPE')) {
      this.pushScene(new PauseScene())
    }
  }
}
```

### Blocking Behaviour

```js
// Pause overlay — blocks updates below but renders on top
class PauseScene extends Scene {
  constructor() {
    super()
    this.blocksUpdateBelow = true
    this.blocksRenderBelow = false
  }
}
```

## Sprites & ECS

### Sprites Are Data, Not Objects

Sprites no longer have `update()` or `render()` — use systems:

```js
// ❌ Old pattern (v0.4.0)
sprite.update(dt)
sprite.render(ctx)

// ✅ v0.5.0
movementSystem.updateOne(sprite, dt)
renderSystem.renderOne(ctx, sprite)
animationSystem.updateOne(sprite, dt)
```

### Build Custom Entities

Any object with the right components works with built-in systems:

```js
const particle = {
  transform: new Transform(x, y),
  collider: new Collider(4, 4),
  renderable: new Renderable(null, { fill: '#ff0', shape: 'circle' }),
  velocity: new Vec2(0, -100),
  visible: true,
}

movementSystem.updateOne(particle, dt)
renderSystem.renderOne(ctx, particle)
```

### Use `ActivePool` for Frequent Spawning

```js
const bulletPool = new ActivePool({
  create: () => new Sprite(0, 0, 4, 4),
  reset: (b) => { b.visible = false; b.velocity.set(0, 0) },
  initialSize: 50,
})

function update(dt) {
  bulletPool.updateActive(b => {
    movementSystem.updateOne(b, dt)
    if (outOfBounds(b)) bulletPool.release(b)
  })
}

function render(ctx) {
  bulletPool.forEachActive(b => renderSystem.renderOne(ctx, b))
}
```

## Groups

### Groups Are Containers Only

Groups no longer have `update` or `render` — iterate manually:

```js
// ✅ v0.5.0
for (const sprite of group) {
  movementSystem.updateOne(sprite, dt)
}
```

### Use `dispose()` for Cleanup

```js
group.dispose()  // unregisters from CollisionSystem + clears
```

### Enable Spatial Hash for Large Groups

```js
group.useSpatialHash(64)
```

### Prefer Callback Style for Group Collisions

```js
// Zero-alloc — no array created
bullets.collideGroup(enemies, (bullet, enemy) => {
  bullet.kill()
  enemy.health--
})

// Array style — allocates
const pairs = bullets.collideGroup(enemies)
```

## Camera

### Set Up a Camera Early

```js
const camera = new Camera(400, 300, 800, 600)  // becomes Camera.main
```

### Follow the Player

```js
scene.update = function (dt) {
  camera.follow(player)
}
```

### Convert Coordinates

```js
const worldPos = {}
camera.screenToWorld(mouseX, mouseY, worldPos)
```

## Animation

### Define Clips with Arrays of Preloaded Images

```js
const frames = await Promise.all([
  ImageLoader.load('walk1.png'),
  ImageLoader.load('walk2.png'),
  ImageLoader.load('walk3.png'),
  ImageLoader.load('walk4.png'),
])

player.animation.add('walk', { frames, fps: 8, loop: true })
```

### Advance Animation in Update

```js
scene.update = function (dt) {
  animationSystem.updateOne(player, dt)
}
```

## Input

### Use Action Bindings Over Raw Keys

```js
// ✅ Good
Input.bind('JUMP', 'SPACE')
Input.bind('JUMP', 'W')
if (Input.justPressed('JUMP')) jump()

// ❌ Avoid
if (Input.justPressed('SPACE') || Input.justPressed('W')) jump()
```

### Action Bindings Replace Manual Remapping for Common Cases

For simple multi-key actions, `bind()` is more ergonomic than `mapKey()`:

```js
Input.bind('FIRE', 'z')
Input.bind('FIRE', 'x')
Input.bind('FIRE', 'ENTER')
```

## Collision

### Centralize with CollisionSystem

`CollisionSystem` manages all collision state and spatial hashes. Use it directly for custom entity arrays:

```js
collisionSystem.useSpatialHash(myEntities, myEntities)
collisionSystem.beginFrame()  // rebuild all hashes
collisionSystem.collideRect(myEntities, rect)
```

### Group Methods Delegate Automatically

```js
group.useSpatialHash(64)  // registers with collisionSystem
group.collideRect(rect)   // delegates to collisionSystem
```

## Game Loop

### Never Put Simulation Logic in `render()`

```js
// ❌ Bad
render(ctx) {
  player.velocity.x = 5     // frame-rate dependent!
  player.render(ctx)
}

// ✅ Good
update(dt) {
  player.velocity.x = 200
  movementSystem.updateOne(player, dt)
}
render(ctx) {
  renderSystem.renderOne(ctx, player)
}
```

### Scene Operations Are Safe During Update

Scene mutations called inside `update()` are deferred and applied after the update cycle:

```js
scene.update = function (dt) {
  if (Input.justPressed('ESCAPE')) {
    this.pushScene(new PauseScene())  // queued, not applied immediately
  }
}
```

## State

### Keep UI State Separate from Game State

```js
const gameState = new State({ score: 0, lives: 3 })
this.selectedOption = 0     // scene-local UI state

gameState.subscribe(s => {
  game.patchUI({ score: `Score: ${s.score}` })
})
```

## Asset Loading

### Preload and Use Progress

```js
class BootScene extends Scene {
  async enter() {
    const task = ImageLoader.loadAll({
      player: 'player.png',
      enemy: 'enemy.png',
    })
    task.onProgress((l, t) => console.log(`${Math.round(l/t*100)}%`))
    await task
    this.switchScene(new MenuScene())
  }
}
```

## Performance

### Use `ActivePool` for All Hot Objects

Prefer `ActivePool` over raw `Pool` for frequently created/destroyed objects — it tracks active objects so you never iterate inactive ones.

### Use Callback-Style Collision to Avoid Allocations

```js
group.collideGroup(other, (a, b) => { ... })  // no array allocated
```

### Enable Spatial Hash for Groups > 50 Entities

### Use Pool-Friendly Methods

```js
Vec2.lerpInto(out, a, b, t)     // no allocation
rect.getCenter(out)             // no allocation
group.collideRect(rect, out)    // reuse output array
```

### Set Up Camera for Viewport Culling

Camera-based rendering automatically culls off-screen entities.
