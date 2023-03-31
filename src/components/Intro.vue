<template>
  <div class="menu" v-if="!start">
    <div class="head-margin"></div>
    <div>
      <h1>Complete the Level</h1>
      <div class="menu-options">
        <div class="menu-option" @click="start = true">
          Start Game
        </div>
        <div class="menu-option">
          How to Play
        </div>
      </div>
    </div>

    <div class="bottom-margin"></div>
  </div>
  <Game v-else />
</template>

<script>
import Game from './Game.vue'
export default {
  name: 'menu',
  components: { Game },
  data () {
    return {
      start: false
    }
  }
}
</script>

<style>
.menu-wrapper {
  display: flex;
  justify-content: center;
  width: 100vw;
  height: 100vh;
  background: linear-gradient(#ffffff, #f3f7df)
}

.head-margin {
  height: 8rem;
  width: 100vw;
  background: #655643
}

</style>
