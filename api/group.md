# Group

The `Group` class is a pure iterable container for sprites. It does NOT own `update` or `render` logic — use systems directly. Collision queries delegate to `CollisionSystem`.

## Constructor

```js
const group = new Group()
```

## Sprite Collection

| Method | Description |
|--------|-------------|
| `add(sprite)` | Adds a sprite (no duplicates). Pushes `this` to `sprite.groups`. |
| `remove(sprite)` | Removes a sprite and its group reference. |
| `has(sprite)` | Returns `boolean`. |
| `clear()` | Removes all sprites and cleans up group references. |
| `dispose()` | Unregisters from `CollisionSystem` and calls `clear()`. |
| `length` (getter) | Returns the number of sprites. |
| `[Symbol.iterator]()` | Makes the group iterable (`for...of`, spread, etc.) |

```js
for (const sprite of group) {
  movementSystem.updateOne(sprite, dt)
  renderSystem.renderOne(ctx, sprite)
}
```

## Spatial Hash

Enable spatial hashing for accelerated collision detection:

```js
group.useSpatialHash(cellSize)  // returns this for chaining
```

Delegates to `collisionSystem.useSpatialHash(this, this._sprites, cellSize)`.

## Collision Queries

All methods skip invisible sprites and accept an optional `out` array for pool-friendly reuse. All delegate to `CollisionSystem`.

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `collideRect(rect, out?)` | `collideRect(rect)` | `Sprite[]` | Sprites whose AABB overlaps the given `Rect` |
| `collidePoint(point, out?)` | `collidePoint({x, y})` | `Sprite[]` | Sprites whose AABB contains the point |
| `collideGroup(other, out?)` | `collideGroup(otherGroup)` | `[spriteA, spriteB][]` | All colliding pairs between two groups |
| `collideSprite(sprite, out?)` | `collideSprite(sprite)` | `Sprite[]` | Sprites colliding with the given sprite |

```js
const hits = group.collideRect(rect)
hits.forEach(s => s.kill())
```

`collideGroup` also accepts a callback for zero-alloc collision processing:

```js
bullets.collideGroup(enemies, (bullet, enemy) => {
  bullet.kill()
  enemy.health--
})
```

## Array Utilities

| Method | Description |
|--------|-------------|
| `forEach(fn)` | `_sprites.forEach(fn)` |
| `filter(fn)` | `_sprites.filter(fn)` |
| `map(fn)` | `_sprites.map(fn)` |

```js
group.forEach(s => s.health -= 1)
```

## Example

```js
import { Scene, Group, Sprite, movementSystem, renderSystem } from 'jygame'

const enemies = new Group()

for (let i = 0; i < 10; i++) {
  const enemy = new Sprite(i * 60, 50, 32, 32)
  enemy.style.fill = '#e8590c'
  enemies.add(enemy)
}

enemies.useSpatialHash(64)

const scene = new Scene()
scene.update = function (dt) {
  for (const sprite of enemies) {
    movementSystem.updateOne(sprite, dt)
  }
  const hits = enemies.collideRect(playerRect)
  hits.forEach(s => s.kill())
}

scene.render = function (ctx) {
  for (const sprite of enemies) {
    renderSystem.renderOne(ctx, sprite)
  }
}
```
