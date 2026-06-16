# Audio Definitions

Declarative audio configuration via `AudioDefinition`, loading via `AudioLoader`, and spatial listener via `AudioListener`.

## AudioDefinition

A declarative configuration object that describes how a sound should be loaded and played. Registered via `AudioManager.define(key, config)`.

### Constructor

```js
const def = new AudioDefinition(config)
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `source` | `string` | *(required)* | Path to the audio file |
| `group` | `string` | `"master"` | Group name |
| `volume` | `number` | `1` | Default volume 0–1 |
| `loop` | `boolean` | `false` | Loop by default |
| `maxInstances` | `number` | `32` | Max concurrent instances |
| `spatial` | `boolean` | `false` | Enable spatial audio by default |
| `minDistance` | `number` | `32` | Spatial minimum distance |
| `maxDistance` | `number` | `512` | Spatial maximum distance |
| `attenuation` | `string` | `"linear"` | Attenuation model |

```js
audio.define('explosion', {
  source: 'sounds/explosion.mp3',
  group: 'sfx',
  volume: 0.8,
  maxInstances: 5,
  spatial: true,
  minDistance: 64,
  maxDistance: 300,
})
```

---

## AudioLoader

See the dedicated [AudioLoader page](./audio-loader) for full API details. Quick summary:

- `load(path)` — load an `HTMLAudioElement`
- `loadBuffer(path, audioContext)` — load an `AudioBuffer`
- `loadAll(map)` — batch load via `LoadingTask`
- `get(key)` / `getBuffer(key)` — retrieve cached assets

---

## AudioListener

Represents the listener position for spatial audio. Used by `AudioManager` to compute per-instance attenuation.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x` | `number` | `0` | Listener world X position |
| `y` | `number` | `0` | Listener world Y position |

```js
audio.listener.x = player.transform.x
audio.listener.y = player.transform.y
```

---

## Attenuation

Constants and utility function for spatial audio attenuation.

### Constants

| Constant | Value |
|----------|-------|
| `ATTENUATION_LINEAR` | `"linear"` |
| `ATTENUATION_QUADRATIC` | `"quadratic"` |
| `ATTENUATION_INVERSE` | `"inverse"` |

### `computeAttenuation(distance, minDistance, maxDistance, model, inverseRolloff)`

```js
import { computeAttenuation, ATTENUATION_LINEAR } from 'jygame'

const factor = computeAttenuation(100, 32, 256, ATTENUATION_LINEAR, 4)
```

Computes an attenuation factor (0–1) given the distance parameters.

## Usage

```js
import { AudioManager, AudioLoader, WebAudioBackend } from 'jygame'

const audio = new AudioManager({ backend: new WebAudioBackend() })
const ctx = new AudioContext()

// Load audio buffers
await AudioLoader.loadBuffer('sounds/ambient.wav', ctx)
await AudioLoader.loadBuffer('sounds/footstep.wav', ctx)

// Declare definitions
audio.define('ambient', {
  source: 'sounds/ambient.wav',
  group: 'ambient',
  volume: 0.5,
  loop: true,
})

audio.define('footstep', {
  source: 'sounds/footstep.wav',
  group: 'sfx',
  volume: 0.7,
  maxInstances: 3,
  spatial: true,
})

// Play spatial footsteps
audio.play('footstep', { x: player.x, y: player.y })

// Update listener position each frame
audio.listener.x = camera.x
audio.listener.y = camera.y
```
