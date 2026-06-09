# RenderSystem

The `RenderSystem` handles drawing entities to the Canvas 2D context. It applies the current `Camera` transform, performs viewport culling, and delegates drawing to each entity's `Renderable`.

A singleton `renderSystem` instance is exported and used internally by various workflows.

## Constructor

```js
new RenderSystem()
```

## Methods

### `render(ctx, entities, camera?)`

```js
renderSystem.render(ctx, entities)
renderSystem.render(ctx, entities, camera)
```

Renders all visible entities. Steps:
1. Uses the provided camera, `Camera.main`, or identity (no transform) if none
2. Derives view bounds from the camera for culling
3. Applies the camera transform to the context
4. Iterates entities, culling those outside the viewport
5. Draws each entity via `_drawEntity`

### `renderOne(ctx, entity, camera?)`

```js
renderSystem.renderOne(ctx, entity)
renderSystem.renderOne(ctx, entity, camera)
```

Renders a single entity with the same camera/culling logic.

## Camera Integration

`RenderSystem` accepts a `Camera` instance instead of a generic viewport rect:

```js
const camera = new Camera(400, 300, 800, 600)
renderSystem.render(ctx, allEntities, camera)
renderSystem.renderOne(ctx, player, camera)
```

If no camera is provided, it falls back to `Camera.main`. If no camera exists at all, entities render without any viewport transform (identity).

## Viewport Culling

The system derives visible world bounds from the camera's position, zoom, and viewport size. Entities entirely outside these bounds are skipped. For rotated/scaled entities, a conservative bounding radius is used.

## Drawing

### `_drawEntity(ctx, entity)`

Internal method that applies the entity's transform (translate, rotate, scale) and calls `entity.renderable.draw(ctx, collider.width, collider.height)`.

## Default Singleton

```js
import { renderSystem, Camera } from 'jygame'

const camera = new Camera(0, 0, 800, 600)
Camera.setMain(camera)

// RenderSystem uses Camera.main automatically
renderSystem.render(ctx, allEntities)
renderSystem.renderOne(ctx, player)
```

## Standalone Usage

```js
import { renderSystem, Camera, Sprite } from 'jygame'

const camera = new Camera(400, 300, 800, 600)
camera.follow(player)

// Each frame
ctx.clearRect(0, 0, 800, 600)
renderSystem.render(ctx, allEntities, camera)
```
