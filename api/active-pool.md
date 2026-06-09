# ActivePool

An advanced object pool with O(1) acquire/release, active object tracking, batch operations, and instrumentation for tuning pool sizes. Built on top of `Pool`.

The key difference from `Pool`: `ActivePool` tracks which objects are currently **acquired** (in use), provides direct iteration over active objects, and offers convenience methods for batch processing and cleanup.

## Constructor

```js
const pool = new ActivePool(options)
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `create` | `function` | *(required)* | Factory function that returns a new object |
| `reset` | `function` | `() => {}` | Called on each object when released back to the pool |
| `initialSize` | `number` | `0` | Number of objects to pre-allocate |
| `maxSize` | `number` | `Infinity` | Maximum number of released objects to keep |

```js
const bulletPool = new ActivePool({
  create: () => new Sprite(0, 0, 4, 4),
  reset: (b) => { b.visible = false; b.velocity.set(0, 0) },
  initialSize: 50,
})
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `activeCount` | `number` | Number of objects currently acquired (in use) |
| `freeCount` | `number` | Number of objects waiting in the free pool |
| `capacity` | `number` | Total managed objects: active + free |
| `size` | `number` | Alias for `freeCount` |
| `totalCreated` | `number` | Total objects ever allocated (debugging / pool pressure) |
| `peakActive` | `number` | Highest `activeCount` ever reached |
| `peakCapacity` | `number` | Highest `capacity` ever reached |
| `peakFree` | `number` | Highest `freeCount` ever reached (tuning) |
| `activeObjects` | `object[]` | Direct read-only reference to active objects array |

## Methods

### Core Operations

#### `acquire(...args)`

```js
pool.acquire()  // returns pooled or newly created object
```

Acquires an object. Reuses a free one if available; otherwise creates a new one. Marks the object as active.

#### `release(obj)`

```js
pool.release(obj)  // returns boolean (true on success)
```

O(1) release. Resets the object and returns it to the free pool. Returns `true` if the object was active, `false` if already released or not from this pool.

#### `isActive(obj)`

```js
pool.isActive(obj)  // boolean
```

Checks whether an object is currently acquired from this pool.

### Batch Operations

#### `acquireMany(count, out?, init?)`

```js
const arr = pool.acquireMany(100)
pool.acquireMany(100, tempArray)
pool.acquireMany(100, tempArray, (obj, i) => { obj.x = i * 10 })
```

Batch-acquires `count` objects. If `out` is provided, fills it (no allocation). The optional `init` callback is called immediately after each acquire with `(obj, index)`.

#### `releaseMany(objects)`

```js
pool.releaseMany(activeBullets)  // returns number released
```

Releases multiple objects from any iterable. Returns the count of successfully released objects.

### Iteration

#### `forEachActive(fn)`

```js
pool.forEachActive((obj, index) => { obj.render(ctx) })
```

Iterates all active objects. Safe for read/render.

#### `updateActive(fn)`

```js
pool.updateActive((obj, index) => { obj.update(dt) })
```

Iterates all active objects for mutation. Semantically identical to `forEachActive` — use to clarify intent.

### Cleanup

#### `releaseInactive(predicate)`

```js
pool.releaseInactive(obj => outOfBounds(obj))
```

Releases every active object matching the predicate. Iterates backward for safe removal.

#### `clearActive()`

```js
pool.clearActive()
```

Releases every active object at once. Fast scene cleanup — resets all, then clears the active array in one step.

### Warmup

#### `warmup(count)`

```js
pool.warmup(50)
```

Pre-creates `count` objects and adds them to the free pool. Use during loading screens to avoid mid-game allocation spikes.

## Usage

```js
import { ActivePool, Sprite } from 'jygame'

const bulletPool = new ActivePool({
  create: () => new Sprite(0, 0, 8, 8),
  reset: (b) => { b.visible = false; b.velocity.set(0, 0) },
  initialSize: 30,
})

// During gameplay
function fire() {
  const b = bulletPool.acquire()
  b.transform.x = playerX
  b.transform.y = playerY
  b.velocity.set(0, -300)
  b.visible = true
}

// Each frame — update only active bullets
function update(dt) {
  bulletPool.updateActive(b => {
    b.update(dt)
    if (outOfBounds(b)) bulletPool.release(b)
  })
}

// Render only active bullets
function render(ctx) {
  bulletPool.forEachActive(b => renderSystem.renderOne(ctx, b))
}
```
