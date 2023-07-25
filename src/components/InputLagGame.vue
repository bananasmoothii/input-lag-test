<script lang="ts">
import {defineComponent} from 'vue';
import * as Collections from 'typescript-collections';

type SquareTimes = { startTime: number, endTime: number };

type DataType = {
  inputLagMillisList: Collections.Queue<number>,
  speed: number, // in pixels per second
  squares: Collections.LinkedList<SquareTimes>,
  userSquares: Collections.LinkedList<SquareTimes>,
  isKeyDown: boolean,
  squareWidth: number,
  squareHeight: number,
  squarePadTop: number,
}

export default defineComponent({
  name: "InputLagGame",

  data(): DataType {
    return {
      inputLagMillisList: new Collections.Queue<number>(),
      speed: 200,
      squares: new Collections.LinkedList<SquareTimes>(),
      userSquares: new Collections.LinkedList<SquareTimes>(),
      isKeyDown: false,
      squareWidth: 100,
      squareHeight: 30,
      squarePadTop: 10,
    }
  },

  computed: {
    cursorX(): number {
      return window.innerWidth / 3;
    },
    speedPixelsPerMillis(): number {
      return this.speed / 1000;
    },
    squareWidthTimeMillis(): number {
      return this.squareWidth / this.speedPixelsPerMillis;
    },
    /**
     * Returns the average of inputLagMillisList, or '??' if the list is empty.
     */
    inputLagMillisString(): string {
      const times = this.inputLagMillisList;
      if (times.isEmpty()) return "??";
      let sum = 0;
      times.forEach(time => {
        sum += time;
      });
      return (sum / times.size()).toFixed(2);
    },
  },

  mounted() {
    // Obtain a reference to the canvas element using its id.
    const htmlCanvas = document.getElementById("game-canvas") as HTMLCanvasElement;
    // Obtain a graphics context on the canvas element for drawing.
    const context = htmlCanvas.getContext("2d") as CanvasRenderingContext2D;

    this.resizeCanvas();
    window.addEventListener("resize", this.resizeCanvas);

    document.onkeydown = this.onKeyDown;
    document.onkeyup = this.onKeyUp;
    document.onmousedown = this.onKeyDown;
    document.onmouseup = this.onKeyUp;

    this.update(context);
  },

  watch: {
    speed() {
      this.inputLagMillisList.clear();
      this.squares.clear();
    },
  },

  methods: {
    resizeCanvas() {
      const htmlCanvas = document.getElementById("game-canvas") as HTMLCanvasElement;
      htmlCanvas.width = document.getElementById("game-canvas-container")!.clientWidth;
      const newHeight = window.innerHeight - htmlCanvas.getBoundingClientRect().top - 20;
      if (newHeight > 250) htmlCanvas.height = newHeight;
      else htmlCanvas.height = 250;
    },

    update(ctx: CanvasRenderingContext2D) {

      const canvasWidth = ctx.canvas.width;
      const canvasHeight = ctx.canvas.height;
      const now = Date.now();

      ctx.clearRect(0, 0, canvasWidth, canvasHeight)

      // generate black squares
      const squareWidthTimeMillis = this.squareWidthTimeMillis;
      if (this.squares.isEmpty()) {
        this.squares.add({
          startTime: now,
          endTime: now + squareWidthTimeMillis,
        });
      }
      let lastEndTime = this.squares.last()!.endTime;
      while (this.timeToCoord(now, lastEndTime) < canvasWidth) {
        this.squares.add({
          startTime: lastEndTime + squareWidthTimeMillis,
          endTime: lastEndTime + 2 * squareWidthTimeMillis,
        });
        lastEndTime += 2 * squareWidthTimeMillis;
      }
      // Remove squares that are off the screen.
      while (this.timeToCoord(now, this.squares.first()!.endTime) < 0) {
        this.squares.removeElementAtIndex(0);
      }
      while (this.userSquares.size() > 1 && this.timeToCoord(now, this.userSquares.first()!.endTime) < 0) {
        this.userSquares.removeElementAtIndex(0);
      }

      // Draw a vertical line in the middle: the "cursor".
      ctx.beginPath();
      ctx.moveTo(this.cursorX, 0);
      ctx.lineTo(this.cursorX, canvasHeight);
      ctx.strokeStyle = "grey";
      ctx.stroke();

      let squareOnCursor = false;
      const squareHeight = this.squareHeight;

      // Draw the black squares
      // Stop drawing them when they reach the right edge of the canvas.
      ctx.fillStyle = "#1F1D1A";
      this.squares.forEach(square => {
        const startX = this.timeToCoord(now, square.startTime);
        const endX = this.timeToCoord(now, square.endTime);
        if (startX <= this.cursorX && this.cursorX <= endX) {
          squareOnCursor = true;
        }
        ctx.fillRect(startX, this.squarePadTop, endX - startX, squareHeight);
      });

      // Draw the user's squares
      ctx.fillStyle = "#FFB81F";
      this.userSquares.forEach(square => {
        const startX = this.timeToCoord(now, square.startTime);
        const endX = square.endTime ? this.timeToCoord(now, square.endTime) : this.cursorX;
        ctx.fillRect(startX, this.squarePadTop * 2 + squareHeight, endX - startX, squareHeight);
      });

      if (squareOnCursor) {
        ctx.fillStyle = "#1f1d1a10";
        ctx.fillRect(0, 0, canvasWidth, canvasHeight);
      }

      window.requestAnimationFrame(() => this.update(ctx));
    },

    timeToCoord(now: number, time: number): number {
      // cursorX is at now
      // canvas' left edge (x=0) is at now - (cursorX / speed)
      // t*a + b = x (the relation must be affine, aka linear with offset)
      // we deduce a = speed, b = cursorX - now*speed
      // conclusion: x = t*speed + cursorX - now*speed = (t - now)*speed + cursorX
      return (time - now) * this.speedPixelsPerMillis + this.cursorX;
    },

    onKeyDown() {
      if (this.isKeyDown) return;
      this.isKeyDown = true;
      const now = Date.now();
      this.userSquares.add({
        startTime: now,
        endTime: 0,
      });

      // find the closest black square start time
      let bestDifference = Infinity;
      this.squares.forEach(square => {
        const difference = Math.abs(square.startTime - now);
        if (difference < bestDifference) {
          bestDifference = difference;
        }
      });
      this.inputLagMillisList.add(bestDifference);
    },

    onKeyUp() {
      this.isKeyDown = false;
      const lastSquare = this.userSquares.last();
      if (lastSquare === undefined) return;

      const now = Date.now();
      lastSquare.endTime = now;

      // find the closest black square end time
      let bestDifference = Infinity;
      this.squares.forEach(square => {
        const difference = Math.abs(square.endTime - now);
        if (difference < bestDifference) {
          bestDifference = difference;
        }
      });
      this.inputLagMillisList.add(bestDifference);
    },

    clearInputLagMillisList() {
      this.inputLagMillisList.clear();
      setTimeout(() => {
        this.inputLagMillisList.clear();
        document.getElementById("speed-slider")!.focus();
      }, 1000);
    },
  }
});
</script>

<template>
  <div class="d-flex justify-content-center speed-config-container">
    <label class="form-label text-nowrap" for="speed-slider">Speed: {{ speed }}</label>
    <input id="speed-slider" v-model="speed" class="form-range" max="1000" min="10" step="1" type="range">
  </div>

  <div class="d-flex justify-content-between align-items-center my-3">
    <div class="d-flex align-items-center">
      <h3 class="me-2">Average input lag: <strong>{{ inputLagMillisString }} ms</strong></h3>
      <small class="based-on">
        (Based on the last {{ inputLagMillisList.size() }} inputs)
      </small>
    </div>
    <button class="btn btn-primary" @click="clearInputLagMillisList">Clear</button>
  </div>
  <div id="game-canvas-container">
    <canvas id="game-canvas"/>
  </div>
</template>

<style lang="scss" scoped>
@import "@/scss/custom";

.speed-config-container {
  gap: map-get($spacers, 2);
}
</style>
