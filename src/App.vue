<template>
<div>
  <GameOfLive ref="game" :densitiy="densitiy" :speed="speed" :running="running"/>

  <div id="menu">
    <input type="number" v-model="speed" placeholder="speed"/>
    <input type="range" min="1" max="100" :value="densitiy" @change="(e) => {densitiy = parseInt(e.target.value)}">
    <button type="button" @click="() => {running = true}" id="start" >start</button>
    <button type="button" @click="reset" id="stop" >reset</button>
    <button type="button" @click="() => {running = false}" id="stop" >stop</button>
    <button type="button" @click="random" id="stop" >random</button>
  </div>
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
  },
  data() {
    return {
      densitiy: 100,
      speed: 500,
      running: false,
    }
  },
  methods: {
    random(){
      this.$refs.game.random();
    },
    reset(){
      this.$refs.game.reset();
    }
  }
}
</script>

<style lang="scss">
#menu{
  position: fixed;
  top: 0;
  left: 0;
}
#menu{
  position: fixed;
  top: auto;
  bottom: 10px;
  left: 10px;
  background-color: white;
  padding: 10px;
  &>*{
    margin: 5px;
    float: left;
  }
}
</style>
