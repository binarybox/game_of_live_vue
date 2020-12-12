<template>
<div ref="game" id="game" class="hello">
  <div :key="i" @click="alive(i)"
  v-bind:class="{
    'alive': logic[i],
    /* 'leftT' : logic[i-1] || logic[i - vertical - 1] || logic[i - vertical],
    'leftB' : logic[i-1] || logic[i + vertical - 1] || logic[i + vertical],
    'rightT' : logic[i+1] || logic[i - vertical + 1] || logic[i - vertical],
    'rightB' : logic[i+1] || logic[i + vertical + 1] || logic[i + vertical], */
    }"
  class="box" v-for="i in (horizontal * vertical)"
  :style="{
    height: (scale * ratio) + '%',
    width: scale + '%'
    }"
  />
  <div id="menu">
    <input type="number" v-model="speed" />
    <button type="button" @click="start" id="start" >start</button>
    <button type="button" @click="() => {logic = []}" id="stop" >reset</button>
    <button type="button" @click="stop" id="stop" >stop</button>
    <button type="button" @click="random" id="stop" >random</button>
  </div>
</div>
</template>

<script>
export default
{
  name: 'GameOfLive',
  data () {
    return {
      scale: 1,
      ratio: 1.0,
      logic: [],
      run: null,
      speed: 500,
      input: "",
      stoping: true
    }
  },
  watch: {
    input: function(){
      this.logic = []
      const middleH = Math.floor(this.horizontal / 2);
      const middleV = Math.floor(this.vertical / 2)
      const start = middleV - Math.floor((10 * this.input.length) / 2)
      const upper = this.input.toUpperCase()
      let index = start - 4;
      const row = (i) => {
        return (this.vertical) * (middleH +i)
      }
      for(let i in this.input){
        if(upper[i] == "A"){
          //this.logic[((this.vertical) * (middleH-1) + start)] = true
          for(let j = 0; j <= 7; j++){
            this.logic[(row(0) + index+j)] = true
            this.logic[(row(1) + index+j)] = true
          }
          for(let j = 2; j <= 5; j++){
            this.logic[(row(-5) + index+j)] = true
            this.logic[(row(-4) + index+j)] = true
          }
          for(let j = -3; j <= 4; j++){
            this.logic[(row(j)) + index] = true
            this.logic[(row(j)) + index+1] = true
            this.logic[(row(j)) + index+7] = true
            this.logic[(row(j)) + index+6] = true
          }
          this.logic[(row(-3)) + index + 2] = true
          this.logic[(row(-3)) + index + 5] = true
          this.logic[(row(-4)) + index+1] = true
          this.logic[(row(-4)) + index+6] = true
        }
        if(upper[i] == "B"){
          //this.logic[((this.vertical) * (middleH-1) + start)] = true
          for(let j = -5; j <= 4; j++){
            this.logic[(row(j) + index+0)] = true
            this.logic[(row(j) + index+1)] = true
            this.logic[(row(j) + index+6)] = true
          }
          for(let j = 0; j <= 6; j++){
            this.logic[(row(-5) + index+j)] = true
            this.logic[(row(-4) + index+j)] = true
            this.logic[(row(-1) + index+j)] = true
            this.logic[(row(-0) + index+j)] = true
            this.logic[(row(3) + index+j)] = true
            this.logic[(row(4) + index+j)] = true
          }
          for(let j = -4; j <= 3; j++){
            this.logic[(row(j)) + index+7] = true
          }
          delete this.logic[row(-1) + index+7]
        }
        if(upper[i] == "H"){
          //this.logic[((this.vertical) * (middleH-1) + start)] = true
          for(let j = 0; j <= 7; j++){
            this.logic[(row(0) + index+j)] = true
            this.logic[(row(-1) + index+j)] = true
          }
          for(let j = -5; j <= 4; j++){
            this.logic[(row(j)) + index] = true
            this.logic[(row(j)) + index+1] = true
            this.logic[(row(j)) + index+7] = true
            this.logic[(row(j)) + index+6] = true
          }
        }
        index += 10
      }
    }
  },
  mounted () {
    window.addEventListener('resize', this.resize);
    this.resize()
  },
  computed: {
    horizontal: function(){
      const horizontal = Math.ceil(100 / (this.scale * this.ratio))
      this.logic = []
      return horizontal;
    },
    vertical: function(){
      const vertical = Math.ceil(100 / this.scale)
      this.logic = [];
      return vertical
    },
  },
  methods: {
    random(){
      const length = this.horizontal * this.vertical
      const amount = Math.floor(Math.random() * (length / 4));
      this.logic = []
      for(let i = 0; i < amount; i++){
        this.logic[Math.floor(Math.random() * length-1)] = true;
      }
    },
    resize(event){
      this.ratio = this.$refs.game.clientWidth / this.$refs.game.clientHeight
    },
    alive(index){
      if(this.logic[index]){
        delete this.logic[index]
      }
      else{
        this.logic[index] = true
      }
    },
    start(event){
      this.stoping = false
      this.runGame()
      this.runGame()

    },
    stop(){
      this.stoping = true
      //clearInterval(this.run)
    },
    runGame() {
      if(this.stoping){
        return
      }
        const n = Object.assign([], this.logic)
        const checkResurect = (i) => {
          let counter = 0;
          for(let j = -1; j < 2; j++){
            if(this.logic[i- this.vertical + j]){
              counter += 1;
            }
          }
          for(let j = -1; j < 2; j++){
            if(this.logic[i + this.vertical + j]){
              counter += 1;
            }
          }
          if(this.logic[i-1]){
            counter += 1;
          }
          if(this.logic[i+1]){
            counter += 1;
          }
          if(counter == 3){
            n[i] = true
          }
        }
        const check = (i) => {
          if (this.logic[i]){
            return 1
          }
          if(!n[i]){
            checkResurect(i)
          }
          return 0
        }
        for (let i = 0; i < n.length; i++) {
          if(this.logic[i]){
            let counter = 0

            counter += check(i - 1)

            counter += check(i + 1)

            counter += check(i - this.vertical)

            counter += check(i - this.vertical + 1)

            counter += check(i - this.vertical - 1)

            counter += check(i + this.vertical)

            counter += check(i + this.vertical + 1)
            counter += check(i + this.vertical - 1)

            if(counter != 2 && counter != 3){
              delete n[i]
            }
          }
        }
        this.logic = n;
        setTimeout(this.runGame, this.speed)
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
#game {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    height: 100vh;
    width: 100vw;

}

#menu{
  position: absolute;
  bottom: 10px;
  left: 10px;
  &>div{
    float: left;
  }
}

.box{
  //border-radius: 50%;
  width: calc(1% - 2px);
  float:left;
  transition: background-color 0.2s ease;
  &.alive{
    background-color: lightgrey;
  }
  &.leftT{
    border-top-left-radius: 0;
  }
  &.leftB{
    border-bottom-left-radius: 0;
  }
  &.rightT{
    border-top-right-radius: 0;
  }
  &.rightB{
    border-bottom-right-radius: 0;
  }
}
</style>
