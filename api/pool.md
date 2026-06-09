# Pool

Generic object pool for reusing frequently created and destroyed objects. Objects in the free pool carry an internal `__jygamePooled` flag — treat this as reserved.

For tracking active objects and batch operations, see `ActivePool`.

## Constructor

```js
const pool = new Pool(options)
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `create` | `function` | *(required)* | Factory function that returns a new object |
| `reset` | `function` | `() => {}` | Called on each object when released back to the pool |
| `initialSize` | `number` | `0` | Number of objects to pre-allocate |
| `maxSize` | `number` | `Infinity` | Maximum number of released objects to keep |

```js
const bulletPool = new Pool({
  create: () => new Sprite(0, 0, 4, 4),
  reset: (b) => { b.visible = false; b.velocity.set(0, 0) },
  initialSize: 50,
})
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `size` | `number` | Number of objects currently available in the free pool |
| `capacity` | `number` | Total managed objects (free + in-use). Grows on acquire when pool is empty, resets on `drain()` |

## Methods

### `acquire(...args)`

```js
pool.acquire()        // returns pooled or newly created object
pool.acquire(x, y)    // args forwarded to create()
```

Returns a recycled object if one is available, otherwise calls `create(...args)`.

### `release(obj)`

```js
pool.release(obj)
```

Resets the object and returns it to the free pool. Objects beyond `maxSize` are discarded. Calling `release` on an already-released object is a no-op (guarded by `__jygamePooled` flag).

### `grow(n)`

```js
pool.grow(20)
```

Pre-allocates `n` additional objects.

### `drain()`

```js
pool.drain()
```

Empties the pool. Resets `capacity` to `0`.
