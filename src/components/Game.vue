<template>
  <div>
    <Invetory :items=player.invetory />
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight"></canvas>
  </div>
</template>

<script>
import Invetory from './Invetory.vue'

export default {
  name: 'game',
  data () {
    return {
      canvas: null,
      ctx: null,
      canvasWidth: 800,
      canvasHeight: 600,
      player: { x: 400, y: 550, width: 30, height: 30, jumping: false, yVelocity: 0, xVelocity: 0, speed: 3, friction: 0.8, invetory: [] },
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
            { x: 400, y: 400, width: 100, height: 20, color: '#655643' }
          ],
          goal: { x: 700, y: 200, width: 50, height: 50 },
          items: [
            { x: 500, y: 250, width: 40, height: 25, id: 1 }
          ]
        },
        {
          level: 2,
          platforms: [
            { x: 0, y: 550, width: 800, height: 50 },
            { x: 0, y: 50, width: 200, height: 250 },
            { x: 500, y: 450, width: 100, height: 25 },
            { x: 100, y: 150, width: 100, height: 25 }
          ],
          goal: { x: 700, y: 200, width: 50, height: 50 },
          items: [
            { x: 500, y: 350, width: 40, height: 25 },
            { x: 100, y: 250, width: 75, height: 25 }
          ]
        }
      ],
      keys: {}
    }
  },
  components: { Invetory },
  computed: {
    currentLevelData () {
      return this.levels.find(obj => obj.level === this.currentLevel)
    },
    // currentLevel
    // currentLevelCollidables () {
    //   return this.currentLevelData.platfroms.concat(this.currentLevelData.platfroms, this.currentLevelData.items)
    // },
    currentLevelPlatforms () {
      return this.currentLevelData.platforms
    },
    currentLevelGoal () {
      return this.currentLevelData.goal
    },
    currentLevelItems () {
      return this.currentLevelData.items
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

      // Move player left/right
      if (this.keys['KeyA'] || this.keys['ArrowLeft']) {
        this.player.x -= this.player.speed
      }
      if (this.keys['KeyD'] || this.keys['ArrowRight']) {
        this.player.x += this.player.speed
      }

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
        if (this.objectCollision(item) && this.currentLevelItems.find(levelItem => levelItem.id === item.id)) {
          this.itemPickup(item.id)
        }
      })

      // Check for collisions with goal
      this.objectCollision(this.currentLevelGoal) && this.levelComplete()
    },
    platformCollision (platform, xCollide, yCollide) {
      if (xCollide && yCollide) {
      // Check Collision for each side
        const isTopCollide = this.player.yVelocity > 0 && this.player.y + this.player.height <= platform.y + platform.height / 2
        // const isLeftCollide = this.player.x + this.player.width <= platform.x + platform.width / 2
        const isLeftCollide = this.player.x + this.player.width <= platform.x + platform.width / 2 && this.player.y + this.player.height > platform.y && this.player.y < platform.y + platform.height
        // const isRightCollide = this.player.x + this.player.width >= platform.x - platform.width / 2
        const isRightCollide = this.player.x < platform.x + platform.width / 2 && this.player.x + this.player.width > platform.x + platform.width / 2 && this.player.y + this.player.height > platform.y + platform.height
        const isBottomCollide = this.player.yVelocity < 0 && this.player.y >= platform.y + platform.height / 2

        if (isTopCollide) {
          this.player.y = platform.y - this.player.height
          this.player.jumping = false
          this.player.yVelocity = 0
        } else if (isLeftCollide) {
          this.player.x = platform.x - this.player.width
        } else if (isRightCollide) {
          this.player.x = platform.x + platform.width
        } else if (isBottomCollide) {
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
    itemPickup (id) {
      // Remove from level & add to player invetory
      console.log('tis true in this function' + id)
      const getItem = this.currentLevelItems.find(item => item.id === id)
      let removeIndex = this.currentLevelItems.map(item => item.id === id)
      ~removeIndex && this.currentLevelItems.splice(removeIndex, 1)
      this.player.invetory.push(getItem)
    },
    levelComplete () {
      this.player.yVelocity = 0
      this.player.x = 0
      alert('congrats you win')
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
      this.ctx.fillStyle = 'blue'
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
      if (this.drawElement && this.player.invetory.length > 0) {
        const item = this.player.invetory[0]

        // Get coordinates of player click
        const x = this.playerClickX - this.canvas.offsetLeft
        const y = this.playerClickY - this.canvas.offsetTop

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
      this.keys[event.code] = true
      if (event.code === 'Space' && !this.player.jumping) {
        this.player.jumping = true
        this.player.yVelocity = -20
      }
      if (event.code === 'd') {
        this.player.xVelocity = +3
      }
      if (event.code === 'a') {
        this.player.xVelocity = -3
      }
    })

    this.canvas.addEventListener('click', event => {
      console.log('here')
      if (this.player.invetory.length !== 0) {
        console.log('into here')
        this.drawElement = true
        this.playerClickX = event.clientX
        this.playerClickY = event.clientY
      }
    })

    window.addEventListener('keyup', event => {
      this.keys[event.code] = false
    })

    this.gameLoop()
  }
}
</script>
