<template>
  <div>
    <canvas
      id="game-canvas"
      :width="width"
      :height="height"
      @click="handleClick"
    />
    <div class="d-flex justify-content-between">
      <button @click="step" class="btn btn-primary btn-raised">step</button>
      <button @click="start" class="btn btn-primary btn-raised">
        <span v-if="running">stop</span><span v-else>start</span>
      </button>
      <button @click="clear" class="btn btn-danger btn-raised">clear</button>
      <button @click="random" class="btn btn-primary btn-raised">random</button>
      <input type="number" class="form-controll" v-model="fieldSize" />
      <input type="number" class="form-controll" v-model="time" />
      <div>
        <input @input="drawText" class="form-controll" type="text" />
        <button
          class="form-controll btn btn-primary btn-raised"
          @click="readText"
        >
          readText
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "GameOfLive",
  props: {
    width: {
      type: Number,
      default: 1100,
    },
    height: {
      type: Number,
      default: 400,
    },
  },
  data() {
    return {
      ctx: null,
      canvas: null,
      field: [],
      oldField: [],
      fieldSize: 5,
      running: false,
      time: 1000,
    };
  },
  mounted() {
    this.canvas = document.getElementById("game-canvas");
    this.ctx = this.canvas.getContext("2d");
  },
  computed: {
    length: function () {
      return this.width / this.fieldSize;
    },
  },
  methods: {
    drawText(event) {
      const text = event.target.value;

      this.ctx.font = "30px Arial";
      this.ctx.fillText(text, 10, 50);
    },
    readText(e) {
      const image = this.ctx.getImageData(0, 0, this.width, this.height);
      this.ctx.clearRect(0, 0, this.width, this.height);
      for (let x = 0; x < image.data.length; x += 4) {
        if (
          image.data[x] ||
          image.data[x + 1] ||
          image.data[x + 2] ||
          image.data[x + 3]
        ) {
          this.drawOnPoint(x / 4);
        }
      }

      this.drawFromArray(this.field);
    },
    animate(i = 20) {
      if (i >= 10) {
        const j = 2 - i / 10;

        this.fadeOut(j);

        window.requestAnimationFrame(this.animate.bind(null, i - 1));
      } else if (i >= 0) {
        const j = i / 10;

        this.fadeIn(j);
        window.requestAnimationFrame(this.animate.bind(null, i - 1));
      }
    },
    fadeIn(j) {
      const length = this.width / this.fieldSize;
      for (let box in this.field) {
        if (this.field[box]) {
          const x = box % length;
          const y = (box - x) / length;

          if (!this.oldField[box]) {
            this.ctx.fillStyle =
              "rgb( " + 255 * j + "," + 255 * j + "," + 255 * j + ")";
            this.ctx.fillRect(
              x * this.fieldSize,
              y * this.fieldSize,
              this.fieldSize,
              this.fieldSize
            );
          }
        }
      }
    },
    fadeOut(j) {
      this.ctx.clearRect(0, 0, this.width, this.height);
      const length = this.width / this.fieldSize;
      for (let box in this.oldField) {
        const x = box % length;
        const y = (box - x) / length;

        if (!this.field[box] && this.oldField[box]) {
          this.ctx.fillStyle =
            "rgb( " + 255 * j + "," + 255 * j + "," + 255 * j + ")";
          this.ctx.fillRect(
            x * this.fieldSize,
            y * this.fieldSize,
            this.fieldSize,
            this.fieldSize
          );
        } else if (this.oldField[box]) {
          this.ctx.fillStyle = "rgb(0, 0, 0)";
          this.ctx.fillRect(
            x * this.fieldSize,
            y * this.fieldSize,
            this.fieldSize,
            this.fieldSize
          );
        }
      }
    },
    random() {
      this.field = [];
      const density = Math.floor(Math.random() * (this.length * this.length));
      for (let i = 0; i < this.length * this.length; i++) {
        if (Math.random() > 0.8) {
          this.field[i] = true;
        }
      }
      this.drawFromArray(this.field);
    },
    start() {
      if (this.running) {
        this.running = false;
      } else {
        this.readText();
        this.running = true;
        this.step();
      }
    },
    clear() {
      this.field = [];
      this.drawFromArray(this.field);
      this.running = false;
    },
    drawOnPoint(index) {
      const x = Math.floor((index % this.width) / this.fieldSize); // - (e.offsetX % this.fieldSize)
      const y = Math.floor(Math.floor(index / this.width) / this.fieldSize);
      // if ( this.field[ x + ( y * this.length ) ] )
      // {
      //   delete this.field[ x + ( y * this.length ) ]
      // }
      // else
      // {
      this.field[x + y * this.length] = true;

      // }
    },
    handleClick(e) {
      const x = Math.round(e.offsetX / this.fieldSize); // - (e.offsetX % this.fieldSize)
      const y = Math.round(e.offsetY / this.fieldSize);
      const length = this.width / this.fieldSize;

      if (this.field[x + y * length]) {
        delete this.field[x + y * length];
      } else {
        this.field[x + y * length] = true;
      }
      this.drawFromArray(this.field);
    },
    checkBox(box) {
      if (this.oldField[box]) {
        return 1;
      }
      let counter = 0;
      if (this.oldField[box - 1]) {
        counter++;
      }
      if (this.oldField[box + 1]) {
        counter++;
      }
      for (let i = -1; i <= 1; i++) {
        if (this.oldField[box - this.length + i]) {
          counter++;
        }
        if (this.oldField[box + this.length + i]) {
          counter++;
        }
      }
      if (counter === 3) {
        this.field[box] = true;
      }
      return 0;
    },
    step() {
      this.oldField = Array.from(this.field);
      console.log(Object.keys(this.field));
      for (let box in this.field) {
        const idx = parseInt(box);
        let counter = 0;
        counter += this.checkBox(idx + 1);
        counter += this.checkBox(idx - 1);
        for (let i = -1; i <= 1; i++) {
          counter += this.checkBox(idx - this.length + i);
          counter += this.checkBox(idx + this.length + i);
        }
        if (counter !== 2 && counter !== 3) {
          delete this.field[idx];
        }
      }

      this.animate();
      if (this.running) {
        setTimeout(this.step, this.time);
      }
    },
    drawFromArray(field) {
      this.ctx.clearRect(0, 0, this.width, this.height);
      const length = this.width / this.fieldSize;
      for (let box in field) {
        if (field[box]) {
          const x = box % length;
          const y = (box - x) / length;
          this.ctx.fillRect(
            x * this.fieldSize,
            y * this.fieldSize,
            this.fieldSize,
            this.fieldSize
          );
        }
      }
    },
  },
};
</script>

<style>
</style>
