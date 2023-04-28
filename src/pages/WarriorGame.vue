<script setup lang="ts">
import { onMounted } from 'vue'
let canvas: HTMLCanvasElement
let ctx: CanvasRenderingContext2D
let game: Game
let lastTime: number
onMounted(() => {
  Init()
})

// Init
const Init = () => {
  canvas = document.querySelector('.canvasGame') as HTMLCanvasElement
  ctx = canvas.getContext('2d') as CanvasRenderingContext2D
  canvas.width = 1000
  canvas.height = 500
  game = new Game(canvas.width, canvas.height)
  lastTime = 0
  Animate(0)
}

// function
const Animate = (timeStamp: number) => {
  const deltaTime = timeStamp - lastTime
  lastTime = timeStamp
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  game.draw(ctx)
  game.update(deltaTime)
  requestAnimationFrame(Animate)
}

class InputHandler {
  private game
  constructor(game: any) {
    this.game = game
    window.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowLeft') this.game.game.player.direction = 'left'
      else if (e.key === 'ArrowRight') this.game.game.player.direction = 'right'
      if ((e.key === 'ArrowLeft' || e.key === 'ArrowRight') && this.game.game.keys.indexOf(e.key) === -1) {
        this.game.game.keys.push(e.key)
        this.game.game.player.status = 'run'
      } else if (e.key === 'z') {
        this.game.game.player.status = 'attack'
      }  
    })
    window.addEventListener('keyup', (e) => {
      if (this.game.game.keys.indexOf(e.key) > -1) this.game.game.keys.splice(this.game.game.keys.indexOf(e.key), 1)
      this.game.game.player.status = 'idle'
    })
  }
}

class Game {
  private width: number
  private height: number
  private player
  private keys: string[]
 
  constructor(width: number, height: number) {
    this.width = width
    this.height = height
    this.player = new Player(this)
    this.keys = []
  }
  public update(deltaTime: number): void {
    this.player.update(deltaTime)
  }
  public draw(context: any): void {
    this.player.draw(context)
  }
}

class Player {
  private game
  public width: number
  public height: number
  public x: number
  public y: number
  private frameX: number
  private frameY: number
  private maxFrame: number
  private speedX: number
  private input
  private imageIdleRight
  private imageRunRight
  private imageAttackRight
  private imageIdleLeft
  private imageRunLeft
  private imageAttackLeft
  private maxSpeed
  private fps: number
  private maxFps: number
  public status: string
  public direction: string
  constructor(game: any) {
    this.game = game
    this.width = 120
    this.height = 140
    this.x = 0
    this.y = 300
    this.frameX = 0
    this.frameY = 0
    this.maxFrame = 5
    this.speedX = 0.2
    this.input = new InputHandler(this)
    this.imageIdleRight = document.querySelector('.idleRight') as HTMLCanvasElement
    this.imageRunRight = document.querySelector('.runRight') as HTMLCanvasElement
    this.imageAttackRight = document.querySelector('.attackRight') as HTMLCanvasElement
    this.imageIdleLeft = document.querySelector('.idleLeft') as HTMLCanvasElement
    this.imageRunLeft = document.querySelector('.runLeft') as HTMLCanvasElement
    this.imageAttackLeft = document.querySelector('.attackLeft') as HTMLCanvasElement
    this.maxSpeed = 3
    this.fps = 0
    this.maxFps = 8
    this.status = 'idle' // idle、run、attack
    this.direction = 'right' // left、 right
  }
  public update(deltaTime: number) {
    if (this.game.keys.includes('ArrowLeft')) this.speedX = -this.maxSpeed
    else if (this.game.keys.includes('ArrowRight')) this.speedX = this.maxSpeed
    else this.speedX = 0
    this.x = this.speedX + this.x

    // x boundaries
    if (this.x > this.game.width - this.width * 0.5) this.x = this.game.width - this.width * 0.5
    else if (this.x < -this.width * 0.5) this.x = -this.width * 0.5

    // sprite animation
    if (this.status === 'attack') this.maxFrame = 5
    else this.maxFrame = 7
    
    if (this.frameX < this.maxFrame) {
      if(this.maxFps > this.fps) this.fps++
      else {
        this.fps = 0
        this.frameX++
      }
    }
    else this.frameX = 0
  }
  public draw(context: { strokeRect: (arg0: number, arg1: number, arg2: number, arg3: number) => void,
    drawImage: (arg0: any, arg1: number, arg2: number, arg3: number, arg4: number, arg5: number, arg6: number, arg7: number, arg8: number) => void }) {
    let sx = 0
    let x = this.x
    let currentImage = null

    switch (this.frameX) {
      case 0:
        if (this.direction === 'right') sx = (50 * this.frameX) + 80
        else sx = (50 * this.frameX)
        break
      default:
        if (this.direction === 'right') sx = (200 * this.frameX) + 80
        else sx = (200 * this.frameX)
        break
    }

    switch (this.status) {
      case 'idle':
        if (this.direction === 'right') currentImage = this.imageIdleRight
        else  currentImage = this.imageIdleLeft
        break
      case 'run':
        if (this.direction === 'right') currentImage = this.imageRunRight
        else  currentImage = this.imageRunLeft
        break
      case 'attack':
        if (this.direction === 'right') currentImage = this.imageAttackRight
        else  currentImage = this.imageAttackLeft
        break
    }

    if (this.direction === 'right') x += 80 
    context.drawImage(currentImage, sx, this.frameY * this.height, this.width, this.height, x, this.y, this.width, this.height)
  }
}

// class f

</script>

<template lang="pug">
#GameS
  canvas(class="canvasGame")

  //- characters
  img(class="runRight" src="@/assets/images/runRight.png", alt="runRight")
  img(class="idleRight" src="@/assets/images/idleRight.png", alt="idleRight")
  img(class="attackRight" src="@/assets/images/attackRight.png", alt="attackRight")

  img(class="runLeft" src="@/assets/images/runLeft.png", alt="runLeft")
  img(class="idleLeft" src="@/assets/images/idleLeft.png", alt="idleLeft")
  img(class="attackLeft" src="@/assets/images/attackLeft.png", alt="attackLeft")
</template>
  
<style lang="scss" scoped>
#GameS {
  font-family: 'Bangers', cursive;
  & .canvasGame {
    border: 4px solid black;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: black;
    max-width: 100%;
    max-height: 100%;
    box-sizing: border-box;
  }
  & .runRight, .idleRight, .attackRight, .runLeft, .idleLeft, .attackLeft {
    display: none;
  }
}
</style>