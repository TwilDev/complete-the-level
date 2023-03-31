<template>
  <Game
    v-if="start"
    @return="start = false"
    @complete="goToSuccess"
  />
  <div class="menu" :class="[{ 'menu-center' : tutorial }, { 'menu-center' : success }]" v-else>
    <HowToPlay v-if="tutorial" @return="tutorial = false"/>
    <Complete v-else-if="success" @return="success = false"/>
    <div v-else>
      <h1>Complete the Level</h1>
      <div class="menu-options">
        <div class="menu-option" @click="start = true">
          Start Game
        </div>
        <div class="menu-option" @click="tutorial = true">
          How to Play
        </div>
      </div>
    </div>
  </div>

</template>

<script>
import Game from './Game.vue'
import HowToPlay from './HowToPlay.vue'
import Complete from './Complete.vue'

export default {
  name: 'menu',
  components: { Game, HowToPlay, Complete },
  data () {
    return {
      start: false,
      tutorial: false,
      success: false
    }
  },
  methods: {
    goToSuccess () {
      this.start = false
      this.success = true
    }
  }
}
</script>

<style>

.menu {
  height: 100%;
  width: 100%;
  color: white;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.menu-center {
  flex-direction: row!important;
}

.menu-options {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 20px;
}

.menu-option {
  cursor: pointer;
  margin-top: 1rem;
  padding: 1rem 4rem;
  display: inline-block;
}

h1 {
  margin: 0;
  font-weight: 600;
  font-size: 60px;
  font-family: Verdana;
}

.menu-option {
  margin-top: 2rem;
  font-size: 18px;
  background-color: white;
  color: #C86450;
  border: solid 5px white;
}

.menu-option:hover {
  background-color: #C86450;
  transition: background-color .3s ease-in-out;
  color: white;
  transition: background-color duration .2s;
}
</style>
