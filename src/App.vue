<template>
<div>
  <GameOfLive />
</div>
</template>

<script>
import GameOfLive from './components/GameOfLive.vue'

export default
{
  name: 'App',
  components:
  {
    GameOfLive
  }
}
</script>

<style lang="scss">
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
</style>
