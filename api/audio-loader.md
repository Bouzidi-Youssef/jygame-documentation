# AudioLoader

Static utility for loading audio files. Supports both `HTMLAudioElement` (HtmlAudioBackend) and `AudioBuffer` (WebAudioBackend) loading.

## Methods

### `load(path)`

```js
await AudioLoader.load('sounds/click.mp3')
```

Loads an `HTMLAudioElement` and caches it by path. Returns a promise.

### `loadAll(map)`

```js
const task = AudioLoader.loadAll({
  click: 'sounds/click.mp3',
  hit: 'sounds/hit.mp3',
})
await task
```

Loads a map of key-path pairs. Returns a `LoadingTask`.

### `get(key)`

```js
const audio = AudioLoader.get('sounds/click.mp3')
```

Get a cached `HTMLAudioElement` by path or key.

### `has(key)`

Check if a path or key is cached.

### `unload(key)`

Remove a cached entry.

### `clear()`

Clear all caches.

### `loadBuffer(path, audioContext)`

```js
const buffer = await AudioLoader.loadBuffer('sounds/hit.mp3', audioContext)
```

Fetches and decodes an audio file into an `AudioBuffer`. Requires a Web Audio `AudioContext`. Returns a promise.

### `getBuffer(key)`

```js
const buffer = AudioLoader.getBuffer('sounds/hit.mp3')
```

Get a cached `AudioBuffer` by path.

## Usage

```js
import { AudioLoader, AudioManager, WebAudioBackend } from 'jygame'

// For HtmlAudioBackend
await AudioLoader.load('sounds/click.mp3')

// For WebAudioBackend
const ctx = new AudioContext()
await AudioLoader.loadBuffer('sounds/music.wav', ctx)

const audio = new AudioManager({ backend: new WebAudioBackend() })
audio.define('music', { source: 'sounds/music.wav' })
audio.play('music')
```
