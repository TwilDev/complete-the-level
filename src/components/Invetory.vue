<template>
  <div class="inventory-wrapper">
    <h2>Item Held:</h2>
    <h4 class="inventory--no-item" v-if="Object.keys(this.item).length === 0">No items held currently</h4>
    <canvas ref="inventory" :width="canvasWidth" :height="canvasHeight" ></canvas>
  </div>
</template>

<script>
export default {
  props: {
    item: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      canvas: null,
      ctx: null,
      canvasWidth: 300,
      canvasHeight: 50
    }
  },
  methods: {
    drawLoop () {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)
      if (Object.keys(this.item).length !== 0) {
        this.ctx.fillStyle = '#36261e'
        this.ctx.fillRect(95, 0, this.item.width, this.item.height)
      }
      requestAnimationFrame(this.drawLoop)
    }
  },
  mounted () {
    this.canvas = this.$refs.inventory
    this.ctx = this.canvas.getContext('2d')
    this.drawLoop()
  }
}
</script>

<style>
.inventory-wrapper {
  align-items: end;
  flex: 1 1 100%;
  display: flex;
  justify-content: center;
}

.inventory--no-item {
  padding-left: 3rem;
}
</style>
