<template>
  <div class="success-container">
    <span class="go-back" @click="$emit('return')">Back to menu</span>
    <h2>Congratulations!</h2>
    <div class="success-text">
      <p>You were able to complete each level by completing the level with your platform placement...</p>
      <p>Yeah I'm sorry, here's a cookie</p>
      <img src="../assets/cookie.png" alt="goal">
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
.success-container {
  padding: 2rem;
  background: white;
  color: black;
  height: 50%;
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
.success-text {
  margin: 0 2rem;
}

p:last-of-type {
  padding-bottom: 2rem;
}

h2 {
  font-size: 40px;
}
</style>
