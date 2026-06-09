# CollisionSystem

The `CollisionSystem` centralizes all collision detection. It manages spatial hash instances for broad-phase acceleration and provides `collideRect`, `collidePoint`, `collideGroup`, and `collideSprite` queries against entity arrays.

A singleton `collisionSystem` instance is exported and used by `Group` internally.

## Constructor

```js
new CollisionSystem()
```

## Methods

### `useSpatialHash(group, entities, cellSize?)`

```js
collisionSystem.useSpatialHash(group, entities, 64)
```

Registers an entity array (or `Group`) for spatial hashing. The hash is rebuilt each frame via `beginFrame()`. `cellSize` defaults to `64`.

```js
collisionSystem.useSpatialHash(myGroup, myGroup)  // self-managed group
```

### `removeGroup(group)`

```js
collisionSystem.removeGroup(group)
```

Unregisters a group from the collision system. Called automatically by `Group.dispose()`.

### `beginFrame()`

```js
collisionSystem.beginFrame()
```

Rebuilds all registered spatial hashes. Call once per frame before running collision queries, or let the game loop handle it.

### `collideRect(entities, rect, out?)`

```js
collisionSystem.collideRect(entities, rect)       // Sprite[]
collisionSystem.collideRect(entities, rect, out)  // Sprite[]
```

Returns entities whose AABB overlaps the given rect. Uses spatial hash if registered, otherwise does a linear scan.

### `collidePoint(entities, point, out?)`

```js
collisionSystem.collidePoint(entities, { x, y })       // Sprite[]
collisionSystem.collidePoint(entities, { x, y }, out)  // Sprite[]
```

Returns entities whose AABB contains the point.

### `collideGroup(entitiesA, entitiesB, cbOrOut?)`

```js
// Returns array of pairs
const pairs = collisionSystem.collideGroup(groupA, groupB)
pairs.forEach(([a, b]) => { /* ... */ })

// Or use a callback (zero-alloc)
collisionSystem.collideGroup(groupA, groupB, (a, b) => {
  a.kill()
  b.health--
})
```

Returns all colliding entity pairs between two entity arrays. Accepts either an output array or a callback function. When both groups have spatial hashes, uses them for acceleration.

### `collideSprite(entities, sprite, out?)`

```js
collisionSystem.collideSprite(entities, sprite)       // Sprite[]
collisionSystem.collideSprite(entities, sprite, out)  // Sprite[]
```

Returns entities colliding with the given sprite.

## Default Singleton

A pre-created `collisionSystem` instance is used by `Group` automatically:

```js
import { collisionSystem } from 'jygame'

// Group methods delegate here
group.collideRect(rect)      // → collisionSystem.collideRect(this, rect)
group.collideGroup(other)    // → collisionSystem.collideGroup(this, other)
```

## Standalone Usage

```js
import { collisionSystem, Group } from 'jygame'

collisionSystem.useSpatialHash(enemies, enemies)

function update() {
  collisionSystem.beginFrame()
  const hits = collisionSystem.collideRect(enemies, playerRect)
}
```
