<template>
  <div>
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight"></canvas>
  </div>
</template>

<script>
export default {
  data () {
    return {
      canvas: null,
      ctx: null,
      canvasWidth: 800,
      canvasHeight: 600,
      player: { x: 50, y: 50, width: 50, height: 50, jumping: false, yVelocity: 0 },
      platforms: [
        { x: 0, y: 550, width: 800, height: 50 },
        { x: 300, y: 450, width: 200, height: 25 },
        { x: 500, y: 350, width: 100, height: 25 },
        { x: 100, y: 250, width: 100, height: 25 }
      ],
      keys: {}
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
      this.player.yVelocity += 1.5
      this.player.y += this.player.yVelocity
      this.player.jumping = true
      // Check for collisions with platforms
      for (let i = 0; i < this.platforms.length; i++) {
        let platform = this.platforms[i]
        if (this.player.x + this.player.width > platform.x && this.player.x < platform.x + platform.width && this.player.y + this.player.height > platform.y && this.player.y < platform.y + platform.height) {
          this.player.y = platform.y - this.player.height
          this.player.jumping = false
          this.player.yVelocity = 0
        }
      }
      // Check for collisions with screen bounds
      if (this.player.y > this.canvas.height) {
        this.player.yVelocity = 0
        this.player.jumping = false
        this.player.y = 0
      }
      // Move player left/right
      if (this.keys['KeyA'] || this.keys['ArrowLeft']) {
        this.player.x -= 5
      }
      if (this.keys['KeyD'] || this.keys['ArrowRight']) {
        this.player.x += 5
      }
    },
    render () {
      // Clear canvas
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)
      // Draw player
      this.ctx.fillStyle = 'red'
      this.ctx.fillRect(this.player.x, this.player.y, this.player.width, this.player.height)
      // Draw platforms
      this.ctx.fillStyle = 'green'
      for (let i = 0; i < this.platforms.length; i++) {
        let platform = this.platforms[i]
        this.ctx.fillRect(platform.x, platform.y, platform.width, platform.height)
      }
    }
  },
  mounted () {
    // Get canvas context
    this.canvas = this.$refs.canvas
    this.ctx = this.canvas.getContext('2d')

    // Handle keyboard input
    window.addEventListener('keydown', event => {
      this.keys[event.code] = true
      if (event.code === 'Space' && !this.player.jumping) {
        this.player.jumping = true
        this.player.yVelocity = -20
      }
    })
    window.addEventListener('keyup', event => {
      this.keys[event.code] = false
    })

    this.gameLoop()
  }
}
</script>
