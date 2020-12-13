<template>
<div ref="game" id="game" class="hello">
  <div :key="i" @click="alive(i)"
  v-bind:class="{
    'alive': !!logic[i],
    'left' : logic[i-1],
    'right' : logic[i+1],
    'top' : logic[i - vertical],
    'bottom' : logic[i + vertical]
    }"
  :class="'box ' + 'amount-' + logic[i]"
  v-for="i in (horizontal * vertical)"
  :style="{
    height: (scale * ratio) + '%',
    width: scale + '%'
    }"
  />
</div>
</template>

<script>
export default
{
  name: 'GameOfLive',
  props: {
    densitiy: {
        type: Number,
        default: 100
    },
    speed: {
      type: Number,
      default: 1000
    },
    running: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      scale: 1,
      ratio: 1.0,
      logic: [],
      run: null,
      input: "",
      promises: []
    }
  },
  watch: {
    running: {
      handler(running) {
        if(running){
          this.runGame();
        }
      },
      immediate: true
    },
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
    reset(){
      this.logic = []
    },
    random(){
      const length = this.horizontal * this.vertical
      this.logic = []
      for(let i = 0; i < Math.floor(length / (100 /this.densitiy)); i++){
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
        this.logic[index] = 1
      }
    },
    start(event){
      this.stoping = false
      this.runGame()
    },
    stop(){
      this.stoping = true
      //clearInterval(this.run)
    },
    checkBox(i) {
      let counter = 0;
      for(let j = -1; j < 2; j++){
        if(this.logic[i - this.vertical + j]){
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
      return counter === 3 ? 3 : 0
    },
    runGame() {
      if(!this.running){
        return
      }
        const n = []
        const check = (i) => {
          if (!!this.logic[i]){
            return 1
          }
          const count = this.checkBox(i)
          if(count){
            n[i] = count
          }
          return 0
        }
        const p = this.logic.map((e, i) => {
            let counter = 0

            counter += check(i + 1)
            counter += check(i - 1)

            counter += check(i - this.vertical)
            counter += check(i - this.vertical + 1)
            counter += check(i - this.vertical - 1)

            counter += check(i + this.vertical)
            counter += check(i + this.vertical - 1)
            counter += check(i + this.vertical + 1)

            if(counter != 2 && counter != 3){
              return null
            }
            else{
              n[i] = counter
            }
          })

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

.box{
  &:hover{
    background-color: lightgrey;
  }
  border-radius: 50%;
  float:left;
  transition: background-color .5s ease-out;
  &.alive{
    background-color: grey;
  }
  &.left{
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
  }
  &.right{
    border-top-right-radius: 0;
    border-bottom-right-radius: 0;
  }
  &.top{
    border-top-right-radius: 0;
    border-top-left-radius: 0;
  }
  &.bottom{
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;
  }
}
</style>
