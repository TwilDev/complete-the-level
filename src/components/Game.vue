<template>
  <div class="game">
    <div class="game-wrapper">
      <Invetory :item=player.invetory />
      <h2>Level: {{ currentLevel }}</h2>
      <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight" style="background: linear-gradient(#80BCA3, #B6D8C0)"></canvas>
    </div>
  </div>

</template>

<script>
import Invetory from './Invetory.vue'

export default {
  name: 'game',
  components: { Invetory },
  data () {
    return {
      canvas: null,
      ctx: null,
      canvasWidth: 800,
      canvasHeight: 600,
      defaultSpawnx: 100,
      defaultSpawnY: 500,
      player: { x: 100, y: 500, width: 30, height: 30, jumping: false, yVelocity: 0, xVelocity: 0, speed: 4, friction: 0.8, invetory: {} },
      currentLevel: 1,
      drawElement: false,
      itemPlaced: {},
      playerClickX: 0,
      playerClickY: 0,
      levels: [
        {
          level: 1,
          platforms: [
            { x: 0, y: 550, width: 800, height: 50, color: '#655643' },
            { x: 300, y: 520, width: 200, height: 100, color: '#655643' },
            { x: 500, y: 450, width: 100, height: 100, color: '#655643' },
            { x: 360, y: 350, width: 100, height: 20, color: '#655643' }
          ],
          goal: { x: 700, y: 200, width: 50, height: 50, color: '#2880bf' },
          start: { x: 400, y: 550 },
          items: [
            { x: 370, y: 250, width: 80, height: 15, id: 1 }
          ]
        },
        {
          level: 2,
          platforms: [
            { x: 0, y: 550, width: 800, height: 50, color: '#655643' },
            { x: 0, y: 90, width: 200, height: 15, color: '#655643' },
            { x: 500, y: 450, width: 100, height: 25, color: '#655643' },
            { x: 300, y: 350, width: 100, height: 25, color: '#655643' },
            { x: 100, y: 190, width: 100, height: 25, color: '#655643' },
            { x: 600, y: 150, width: 200, height: 20, color: '#655643' }
          ],
          goal: { x: 0, y: 40, width: 50, height: 50, color: '#2880bf' },
          items: [
            { x: 100, y: 450, width: 100, height: 15, id: 1 }
          ]
        },
        {
          level: 3,
          platforms: [
            { x: 0, y: 550, width: 800, height: 50, color: '#655643' },
            { x: 300, y: 0, width: 15, height: 800, color: '#655643' },
            { x: 50, y: 0, width: 15, height: 800, color: '#655643' }
          ],
          goal: { x: 50, y: 130, width: 50, height: 50, color: '#2880bf' },
          items: [
            { x: 100, y: 450, width: 100, height: 15, id: 1 }
          ]
        }
      ],
      keys: {}
    }
  },
  computed: {
    currentLevelData () {
      return this.levels.find(obj => obj.level === this.currentLevel)
    },
    currentLevelPlatforms () {
      return this.currentLevelData.platforms
    },
    currentLevelGoal () {
      return this.currentLevelData.goal
    },
    currentLevelItems () {
      return this.currentLevelData.items
    },
    usableItems () {
      return this.player.invetory
    },
    currentLevelSpawn () {
      return this.currentLevelData().start
    }
  },
  methods: {
    gameLoop () {
      this.update()
      this.render()
      requestAnimationFrame(this.gameLoop)
    },
    update () {
      // Update player position

      // Set initial gravity
      this.player.yVelocity += 1.5
      this.player.y += this.player.yVelocity
      this.player.x += this.player.xVelocity

      // Check collision for placed platforms
      const xCollide = this.player.x + this.player.width > this.itemPlaced.x && this.player.x < this.itemPlaced.x + this.itemPlaced.width
      const yCollide = this.player.y + this.player.height > this.itemPlaced.y && this.player.y < this.itemPlaced.y + this.itemPlaced.height
      this.platformCollision(this.itemPlaced, xCollide, yCollide)

      // Check for collisions with platforms
      for (let platform of this.currentLevelPlatforms) {
        const xCollide = this.player.x + this.player.width > platform.x && this.player.x < platform.x + platform.width
        const yCollide = this.player.y + this.player.height > platform.y && this.player.y < platform.y + platform.height
        this.platformCollision(platform, xCollide, yCollide)
      }

      // Check item collision
      this.currentLevelItems.forEach((item) => {
        if (this.objectCollision(item)) {
          this.itemPickup(item)
        }
      })

      // Check for collisions with goal
      this.objectCollision(this.currentLevelGoal) && this.levelComplete()
    },
    platformCollision (platform, xCollide, yCollide) {
      if (xCollide && yCollide) {
        // Check Collision for each side
        const isTopCollide = this.player.yVelocity > 0 && this.player.y + this.player.height <= platform.y + platform.height / 2
        const isLeftCollide = this.player.x + this.player.width <= platform.x + platform.width / 2 &&
         this.player.y + this.player.height > platform.y &&
         this.player.y < platform.y + platform.height &&
         this.player.y + this.player.height <= platform.y + platform.height
        // const isLeftCollide = this.player.x + this.player.width <= platform.x + platform.width / 2 && this.player.y + this.player.height > platform.y && this.player.y < platform.y + platform.height
        // const isRightCollide = this.player.xVelocity < 0 && this.player.x + this.player.width >= platform.x + platform.width / 2 && this.player.y + this.player.height > platform.y && this.player.y < platform.y + platform.height
        const isRightCollide = this.player.xVelocity < 0 &&
         this.player.x + this.player.width >= platform.x + platform.width / 2 &&
         this.player.y + this.player.height > platform.y &&
         this.player.y < platform.y + platform.height &&
         this.player.y + this.player.height <= platform.y + platform.height
        const isBottomCollide = this.player.yVelocity < 0 && this.player.y >= platform.y + platform.height / 2

        if (isTopCollide) {
          console.log('Top collided')
          this.player.y = platform.y - this.player.height
          this.player.jumping = false
          this.player.yVelocity = 0
        } else if (isLeftCollide) {
          console.log('Left collided')
          this.player.x = platform.x - this.player.width
        } else if (isRightCollide) {
          console.log('Right collided')
          this.player.x = platform.x + platform.width
        } else if (isBottomCollide) {
          console.log('Bottom collided')
          this.player.y = platform.y + platform.height
          this.player.yVelocity = 0
        } else {
          this.player.y = platform.y - this.player.height
          this.player.jumping = false
          this.player.yVelocity = 0
        }
      }
    },
    objectCollision (collidingObject) {
      let isColliding
      const xCollide = this.player.x + this.player.width > collidingObject.x && this.player.x < collidingObject.x + collidingObject.width
      const yCollide = this.player.y + this.player.height > collidingObject.y && this.player.y < collidingObject.y + collidingObject.height
      xCollide && yCollide ? isColliding = true : isColliding = false
      return isColliding
    },
    itemPickup (item) {
      // console.log('got into here with id of ' + id)
      const filteredArray = [...this.currentLevelItems.filter((obj) => {
        return obj.id !== item.id
      })]
      this.levels.find(obj => obj.level === this.currentLevel).items = filteredArray
      console.log(item)
      this.player.invetory = {...item}
    },
    levelComplete () {
      this.player.yVelocity = 0
      this.player.x = this.defaultSpawnx
      this.player.y = this.defaultSpawnY
      alert('congrats you win')
      this.player.xVelocity = 0
      this.playerClickX = null
      this.playerClickY = null
      this.player.invetory = {}
      this.currentLevel++
    },
    render () {
      // Clear canvas
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)

      // Draw platforms
      for (let i = 0; i < this.currentLevelPlatforms.length; i++) {
        let platform = this.currentLevelPlatforms[i]
        this.ctx.fillStyle = platform.color
        this.ctx.fillRect(platform.x, platform.y, platform.width, platform.height)
      }

      // Draw Goals
      this.ctx.fillStyle = this.currentLevelGoal.color
      this.ctx.fillRect(this.currentLevelGoal.x, this.currentLevelGoal.y, this.currentLevelGoal.width, this.currentLevelGoal.height)

      // Draw pickups
      this.ctx.fillStyle = 'orange'
      for (let i = 0; i < this.currentLevelItems.length; i++) {
        let item = this.currentLevelItems[i]
        this.ctx.fillRect(item.x, item.y, item.width, item.height)
      }

      // Draw player
      this.ctx.fillStyle = '#E6AC27'
      this.ctx.fillRect(this.player.x, this.player.y, this.player.width, this.player.height)

      // Draw placed items
      if (this.drawElement && Object.keys(this.player.invetory).length !== 0) {
        const item = this.player.invetory

        // Get coordinates of player click / by 2 to place in the middle of mouse clicka
        const x = (this.playerClickX - this.canvas.offsetLeft) - (item.width / 2)
        const y = (this.playerClickY - this.canvas.offsetTop) - (item.height / 2)

        this.ctx.fillStyle = '#36261e'
        this.ctx.fillRect(x, y, item.width, item.height)
        this.itemsPlaced = null
        this.itemPlaced = {x, y, width: item.width, height: item.height}
      }
    }
  },
  mounted () {
    // Get canvas context
    this.canvas = this.$refs.canvas
    this.ctx = this.canvas.getContext('2d')
    document.body.style.margin = 0

    // Handle keyboard input
    window.addEventListener('keydown', event => {
      //  this.keys[event.code] = true
      if (event.code === 'Space' && !this.player.jumping) {
        console.log('jump!')
        this.player.jumping = true
        this.player.yVelocity = -20
      }
      if (event.code === 'KeyD' || event.code === 'ArrowLeft') {
        this.player.xVelocity = +this.player.speed
      }
      if (event.code === 'KeyA' || event.code === 'ArrowRight') {
        this.player.xVelocity = -this.player.speed
      }
    })

    this.canvas.addEventListener('click', event => {
      if (Object.keys(this.player.invetory).length !== 0) {
        this.drawElement = true
        this.playerClickX = event.clientX
        this.playerClickY = event.clientY
      }
    })

    window.addEventListener('keyup', event => {
      if (event.code === 'KeyD' || event.code === 'ArrowLeft') {
        this.player.xVelocity = 0
      }
      if (event.code === 'KeyA' || event.code === 'ArrowRight') {
        this.player.xVelocity = 0
      }
    })

    this.gameLoop()
  }
}
</script>

<style>
.game-wrapper {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

</style>
