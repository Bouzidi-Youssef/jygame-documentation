# SpatialHash

A spatial partitioning data structure for broad-phase collision detection. Organized as a grid of cells; only entities in the same or adjacent cells are checked against each other.

## Constructor

```js
new SpatialHash(cellSize)
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `cellSize` | `number` | `64` | Width and height of each grid cell in pixels |

## Methods

### `rebuild(entities)`

```js
spatialHash.rebuild(entities)
```

Clears all cells and re-inserts the given array of entities (any objects with `transform`, `collider`, and `visible`). Invisible entities are skipped. Each entity gets `__shId` and `__shStamp` internal fields.

### `collideRect(rect, out?)`

```js
spatialHash.collideRect(rect)       // Entity[]
spatialHash.collideRect(rect, out)  // Entity[]
```

Returns entities whose AABB overlaps the given rect-like object `{ left, right, top, bottom }`. Uses stamp-based duplicate prevention (zero-alloc).

### `collidePoint(point, out?)`

```js
spatialHash.collidePoint({ x, y })      // Entity[]
spatialHash.collidePoint({ x, y }, out) // Entity[]
```

Returns entities whose AABB contains the point.

### `collideGroup(other, cbOrOut?)`

```js
// Returns array of pairs
const pairs = spatialHash.collideGroup(otherHash)
pairs.forEach(([a, b]) => { /* ... */ })

// Or use a callback (zero-alloc)
spatialHash.collideGroup(otherHash, (a, b) => {
  a.kill()
  b.health--
})
```

Returns all colliding entity pairs between two `SpatialHash` instances. Accepts either an output array or a callback function.

### `collideSprite(entity, out?)`

```js
spatialHash.collideSprite(entity)      // Entity[]
spatialHash.collideSprite(entity, out) // Entity[]
```

Returns entities colliding with the given entity (excluding itself). Works with any entity, not just sprites.

## Usage with CollisionSystem

`CollisionSystem` manages spatial hash lifecycle — `beginFrame()` calls `rebuild()` on all registered groups:

```js
collisionSystem.useSpatialHash(enemies, enemies)
collisionSystem.beginFrame()     // rebuilds all hashes
collisionSystem.collideRect(enemies, rect)
```

## Standalone Usage

```js
import { SpatialHash } from 'jygame'

const hash = new SpatialHash(64)
hash.rebuild(allEntities)

const nearby = hash.collideRect({ left: 0, right: 100, top: 0, bottom: 100 })
```
