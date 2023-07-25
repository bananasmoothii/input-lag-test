<script lang="ts">
import {defineComponent} from 'vue';
import * as Collections from 'typescript-collections';

type SquareTimes = { startTime: number, endTime: number };
type SquareCoords = { startX: number, endX: number };

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
      return window.innerWidth / 4;
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
      console.log("Resized canvas to " + htmlCanvas.width + "x" + htmlCanvas.height);
    },

    update(ctx: CanvasRenderingContext2D, lastUpdateTime = 0) {

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
      while (this.timeToCoord(now, this.squares.last()!.endTime) < canvasWidth) {
        this.squares.add({
          startTime: this.squares.last()!.endTime + squareWidthTimeMillis,
          endTime: this.squares.last()!.endTime + 2 * squareWidthTimeMillis,
        });
      }
      // Remove squares that are off the screen.
      while (this.timeToCoord(now, this.squares.first()!.endTime) < 0) {
        this.squares.removeElementAtIndex(0);
      }

      // Draw a vertical line in the middle: the "cursor".
      ctx.beginPath();
      ctx.moveTo(this.cursorX, 0);
      ctx.lineTo(this.cursorX, canvasHeight);
      ctx.strokeStyle = "grey";
      ctx.stroke();

      let squareOnCursor = false;

      // Draw the black squares, each 100px wide and 30px tall.
      // Stop drawing them when they reach the right edge of the canvas.
      ctx.fillStyle = "#1F1D1A";
      this.squares.forEach(square => {
        const startX = this.timeToCoord(now, square.startTime);
        const endX = this.timeToCoord(now, square.endTime);
        if (startX <= this.cursorX && this.cursorX <= endX) {
          squareOnCursor = true;
        }
        //console.log(square.startTime, square.endTime, startX, endX);
        ctx.fillRect(startX, this.squarePadTop, endX - startX, this.squareHeight);
      });

      if (squareOnCursor) {
        ctx.fillStyle = "#1f1d1a10";
        ctx.fillRect(0, 0, canvasWidth, canvasHeight);
      }

      window.requestAnimationFrame(() => this.update(ctx, now));
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
      this.userSquares.add({
        startTime: Date.now(),
        endTime: 0,
      });
    },

    onKeyUp() {
      this.isKeyDown = false;
      const lastSquare = this.userSquares.last();
      if (lastSquare === undefined) return;
      lastSquare.endTime = Date.now();
      if (this.userSquares.size() > 4) {
        this.userSquares.removeElementAtIndex(0);
      }
    },
  }
});
</script>

<template>
  <div class="d-flex justify-content-center speed-config-container">
    <label class="form-label text-nowrap" for="speed-slider">Speed: {{ speed }}</label>
    <input id="speed-slider" v-model="speed" class="form-range" max="1000" min="10" step="1" type="range">
  </div>

  <h3>Average input lag: <strong>{{ inputLagMillisString }} ms</strong></h3>
  <p>
    Based on the last {{ inputLagMillisList.size() }} inputs.
  </p>
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
