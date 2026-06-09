# InputContext

Instance-based input handler using the **Pointer Events API** (unified mouse, touch, and pen), `Map`-based key state, multi-touch support, and **action bindings** for grouping multiple keys under logical names.

The `Game` class creates one automatically — you normally interact through the global `Input` facade.

## Constructor

```js
const ctx = new InputContext(options)
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `swipeThreshold` | `number` | `30` | Minimum distance (px) to trigger a swipe |
| `tapTimeout` | `number` | `300` | Max ms for a pointer down-up sequence to count as a tap |

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `x` | `number` | Latest pointer X coordinate (read-only) |
| `y` | `number` | Latest pointer Y coordinate (read-only) |
| `isPointerDown` | `boolean` | Whether any pointer is currently down (read-only) |
| `pointerCount` | `number` | Number of active pointers (read-only) |
| `buffer` | `string[]` | Key buffer for queued input events |

## Methods

### Lifecycle

| Method | Description |
|--------|-------------|
| `init(target)` | Binds keyboard and pointer listeners to the target element |
| `destroy()` | Removes all listeners and clears all state |

### Key State Queries

| Method | Signature | Description |
|--------|-----------|-------------|
| `isDown(key)` | `isDown('UP')` | Is the key or action currently held? |
| `justPressed(key)` | `justPressed('SPACE')` | Was the key or action pressed this frame? |
| `justReleased(key)` | `justReleased('ENTER')` | Was the key or action released this frame? |

### Key Mapping

| Method | Signature | Description |
|--------|-----------|-------------|
| `mapKey(rawKey, alias)` | `mapKey('z', 'JUMP')` | Maps a raw key to a logical alias |
| `unmapKey(rawKey)` | `unmapKey('z')` | Removes a mapping |
| `setKeyMap(map)` | `setKeyMap({ z: 'JUMP' })` | Replaces the entire key map |
| `resetKeyMap()` | `resetKeyMap()` | Restores the default map |
| `getKeyMap()` | `getKeyMap()` | Returns a copy of the current map |

### Action Bindings

Action bindings let you group multiple keys under one logical action name. For example, bind both `SPACE` and `W` to `JUMP`, then query `isDown('JUMP')`.

| Method | Signature | Description |
|--------|-----------|-------------|
| `bind(action, input)` | `bind('JUMP', 'SPACE')` | Binds a key to an action |
| `unbind(action, input)` | `unbind('JUMP', 'SPACE')` | Removes a key from an action |
| `getBindings(action)` | `getBindings('JUMP')` | Returns all keys bound to an action |
| `clearBindings(action)` | `clearBindings('JUMP')` | Removes all bindings for an action |

```js
ctx.bind('FIRE', 'z')
ctx.bind('FIRE', 'x')
ctx.bind('FIRE', 'ENTER')
ctx.isDown('FIRE')     // true if z, x, or ENTER is held
ctx.justPressed('FIRE')
```

### Pointer API

| Method | Signature | Description |
|--------|-----------|-------------|
| `getPointer(id)` | `getPointer(0)` | Returns pointer data or `null` |
| `getPointers()` | `getPointers()` | Returns iterator over active pointers (zero-alloc) |
| `forEachPointer(fn)` | `forEachPointer(p => ...)` | Iterates over all active pointers |

### Gesture Listeners

| Method | Signature | Description |
|--------|-----------|-------------|
| `onSwipe(callback)` | `onSwipe(dir => ...)` | Register swipe listener, returns unsubscribe |
| `removeSwipe(callback)` | `removeSwipe(fn)` | Remove a specific swipe listener |
| `onTap(callback)` | `onTap(({x, y}) => ...)` | Register tap listener, returns unsubscribe |
| `removeTap(callback)` | `removeTap(fn)` | Remove a specific tap listener |

## Default Key Mappings

| Raw Key | Alias |
|---------|-------|
| `ArrowUp`, `w`, `W` | `UP` |
| `ArrowDown`, `s`, `S` | `DOWN` |
| `ArrowLeft`, `a`, `A` | `LEFT` |
| `ArrowRight`, `d`, `D` | `RIGHT` |
| ` ` (Space) | `SPACE` |
| `Escape` | `ESCAPE` |
| `Enter` | `ENTER` |

## Standalone Usage

```js
import { InputContext } from 'jygame'

const input = new InputContext({ swipeThreshold: 20 })
input.init(document.getElementById('game'))

input.bind('JUMP', 'SPACE')
input.bind('JUMP', 'W')
if (input.justPressed('JUMP')) player.jump()
```
