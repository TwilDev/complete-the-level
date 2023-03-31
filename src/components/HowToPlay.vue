<template>
  <div class="h2p-container">
    <span class="go-back" @click="$emit('return')">Back to menu</span>
    <h2>How To Play</h2>
    <div class="h2p-text">
      <p>The aim of the game is to reach the goal point signified by the blue sqaure</p>
      <img src="../assets/goal.png" alt="goal">
      <p>To reach the end point, unlike a traditional platformer you have to collect the golden platforms to complete the level layout yourself</p>
      <img src="../assets/platform.png" alt="">
      <p>When you've collected a platform it'll show up in your items, using your mouse you can create a new platform to help you reach the goal and complete the level. But you can only place one at a time, a well timed jump and precie placement may be needed in cases.</p>
      <img src="../assets/placed_platform.png" alt="">
    </div>
  </div>
</template>

<script>

export default {
  name: 'howtoplay',
  data () {
    return {
      start: false
    }
  }
}
</script>

<style>
.h2p-container {
  padding: 2rem;
  background: white;
  color: black;
  width: 50%;
  border-radius: 1rem;
  margin: 4rem;
}

.go-back {
  position: absolute;
  top: 0;
  left: 0;
  padding: 2rem;
  cursor: pointer;
  font-weight: 600;
  color: white;
 }

p {
  font-size: 16px;
}
@media (min-width: 768px) {
  .h2p-text {
    margin: 0 8rem;
  }
}
.h2p-text {
  margin: 0 2rem;
}

h2 {
  font-size: 40px;
}
</style>
