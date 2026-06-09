# Scene

The `Scene` class organizes your game into distinct states with lifecycle hooks. Scenes are **single-use** — once exited, they must not be re-entered or re-mounted. Create a new scene instance instead.

Scenes live on a **stack** managed by `Game`. By default, a scene **blocks updates** from scenes below it but does **not block rendering**.

## Constructor

```js
const scene = new Scene()
```

Creates `scene.root` — a `<div>` element used as the DOM UI overlay container. Sets `blocksUpdateBelow = true` and `blocksRenderBelow = false`.

## Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `blocksUpdateBelow` | `boolean` | `true` | When on top, pause `update()` on scenes below |
| `blocksRenderBelow` | `boolean` | `false` | When on top, skip `render()` on scenes below |

## Lifecycle Hooks

All hooks are no-ops by default. Scenes throw if `enter()` or `exit()` is called more than once.

| Hook | Signature | Called When |
|------|-----------|-------------|
| `enter()` | `enter()` | Scene becomes active — throws if called twice |
| `exit()` | `exit()` | Scene is exited — runs all cleanup functions, throws if called twice |
| `pause()` | `pause()` | Scene is paused by a scene pushed on top |
| `resume()` | `resume()` | Scene is resumed after the scene above is popped |
| `update(dt)` | `update(dt)` | Each fixed timestep tick |
| `interpolate(alpha)` | `interpolate(alpha)` | After all fixed updates, before render |
| `render(ctx)` | `render(ctx)` | Each frame — receives the canvas 2D context |
| `renderUI()` | `renderUI()` | Returns an HTML string for the DOM overlay |

```js
const scene = new Scene()

scene.enter = () => {
  this.player = new Sprite(100, 100, 32, 32)
}

scene.update = (dt) => {
  movementSystem.updateOne(this.player, dt)
}

scene.render = (ctx) => {
  ctx.clearRect(0, 0, 800, 600)
  renderSystem.renderOne(ctx, this.player)
}
```

## Scene Stack Transitions

| Method | Description |
|--------|-------------|
| `pushScene(scene)` | Push a scene on top (e.g., pause overlay) |
| `popScene()` | Pop the current scene, resume the one below |
| `replaceScene(scene)` | Replace the current scene in-place |
| `switchScene(scene)` | Reset the entire stack to one scene |
| `transitionTo(scene)` | Alias for `switchScene(scene)` |

```js
// Inside a scene method:
this.pushScene(new PauseScene())
this.popScene()
this.replaceScene(new GameOverScene())
this.switchScene(new MenuScene())
```

## Blocking

When `blocksUpdateBelow = true` (default), scenes below are paused via `pause()` / `resume()`. When `blocksRenderBelow = false` (default), all visible scenes render in order from bottom to top.

```js
// A pause overlay that blocks input but lets the game render underneath
class PauseScene extends Scene {
  constructor() {
    super()
    this.blocksUpdateBelow = true
    this.blocksRenderBelow = false
  }
  renderUI() {
    return '<div class="pause-overlay">PAUSED</div>'
  }
}
```

## DOM UI

The scene's `root` div is appended to the game's DOM overlay when active. `renderUI()` populates it.

```js
scene.renderUI = function () {
  return `<h1>Score: ${this.score}</h1>`
}
```

## Event Listener Cleanup

Methods that auto-clean on `exit()`:

| Method | Signature | Description |
|--------|-----------|-------------|
| `on(target, event, handler)` | `on(el, 'click', fn)` | Registers a DOM listener, pushes cleanup |
| `onSwipe(cb)` | `onSwipe(dir => ...)` | Wraps `Input.onSwipe` with auto-cleanup |
| `onTap(cb)` | `onTap(({x, y}) => ...)` | Wraps `Input.onTap` with auto-cleanup |
| `cleanup(fn)` | `cleanup(() => ...)` | Manually push a function to `_cleanups` |

```js
scene.enter = function () {
  this.on(document, 'keydown', this.handleKey)
}
```

## Single-Use Enforcement

```js
game.run(scene)
// later:
game.run(scene)  // ❌ Error: scene already mounted
game.switchScene(scene)  // ❌ Error: scene already exited
```
