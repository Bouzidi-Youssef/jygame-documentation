# Getting Started

## Installation

```bash
npm install jygame
```

Or import directly from a CDN:

```js
import { Game } from 'https://unpkg.com/jygame@latest/jygame.js'
```

## Your First Game

```js
import { Game, Scene, Sprite, Input, movementSystem, renderSystem } from 'jygame'

const scene = new Scene()

scene.enter = function () {
  this.player = new Sprite(100, 100, 32, 32)
  this.player.style.fill = '#63B44E'
}

scene.update = function (dt) {
  if (Input.isDown('RIGHT')) this.player.velocity.x = 200
  else if (Input.isDown('LEFT')) this.player.velocity.x = -200
  else this.player.velocity.x = 0

  if (Input.isDown('UP')) this.player.velocity.y = -200
  else if (Input.isDown('DOWN')) this.player.velocity.y = 200
  else this.player.velocity.y = 0

  movementSystem.updateOne(this.player, dt)
}

scene.render = function (ctx) {
  ctx.clearRect(0, 0, 800, 600)
  renderSystem.renderOne(ctx, this.player)
}

const game = new Game({
  width: 800,
  height: 600,
})

game.run(scene)
```

## Project Structure

```
my-game/
├── index.html
├── package.json
└── src/
    ├── main.js         # Entry point
    ├── scenes/         # Scene files
    ├── entities/       # Game entities
    └── assets/         # Images, fonts
```
