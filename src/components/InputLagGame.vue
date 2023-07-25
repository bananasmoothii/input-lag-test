<script lang="ts">
import {defineComponent} from 'vue'

export default defineComponent({
  name: "InputLagGame",

  data() {
    return {
      inputLagMillisList: [] as number[],
      speed: 1.5,
      updateCanvas: true,
      gameProgressX: 0,
    }
  },

  computed: {
    /**
     * Returns the average of inputLagMillisList, or '??' if the list is empty.
     */
    inputLagMillisString(): string {
      const time = this.inputLagMillisList;
      if (time.length === 0) return "??";
      return (time.reduce((a: number, b: number) => a + b) / time.length).toFixed(2);
    },
  },

  mounted() {
    // Obtain a reference to the canvas element using its id.
    const htmlCanvas = document.getElementById("game-canvas") as HTMLCanvasElement;
    // Obtain a graphics context on the canvas element for drawing.
    const context = htmlCanvas.getContext("2d") as CanvasRenderingContext2D;

    this.resizeCanvas();
    window.addEventListener("resize", this.resizeCanvas);

    this.update(context);
  },

  unmounted() {
    this.updateCanvas = false
  },

  methods: {
    resizeCanvas() {
      const htmlCanvas = document.getElementById("game-canvas") as HTMLCanvasElement;
      htmlCanvas.width = document.getElementById("game-canvas-container")!.clientWidth;
    },

    update(ctx: CanvasRenderingContext2D) {
      const squareWidth = 100;
      const squareHeight = 30;
      const cursorX = 300;
      const squarePadTop = 10;

      const canvasWidth = ctx.canvas.width;
      const canvasHeight = ctx.canvas.height;

      ctx.clearRect(0, 0, canvasWidth, canvasHeight)
      // fill with darker color if the cursor is on a black square, lighter color if it's on a white square
      ctx.fillStyle = (300 - this.gameProgressX) % (squareWidth * 2) < squareWidth ? "#1F1D1A10" : "#F2F2F210";
      ctx.fillRect(0, 0, canvasWidth, canvasHeight);

      // Draw a vertical line in the middle: the "cursor".
      ctx.beginPath();
      ctx.moveTo(cursorX, 0);
      ctx.lineTo(cursorX, canvasHeight);
      ctx.strokeStyle = "grey";
      ctx.stroke();

      // Draw the black squares, each 100px wide and 30px tall.
      // Stop drawing them when they reach the right edge of the canvas.
      ctx.fillStyle = "#1F1D1AFF";
      for (let x = this.gameProgressX; x < canvasWidth; x += 2 * squareWidth) {
        ctx.fillRect(x, squarePadTop, squareWidth, squareHeight);
      }

      this.gameProgressX -= this.speed;

      window.requestAnimationFrame(() => this.update(ctx))
    },
  }
});
</script>

<template>
  <div class="d-flex justify-content-center speed-config-container">
    <label class="form-label text-nowrap" for="speed-slider">Speed: {{ speed }}</label>
    <input id="speed-slider" v-model="speed" class="form-range" max="4" min="0.1" step="0.1" type="range">
  </div>

  <h3>Average input lag: <strong>{{ inputLagMillisString }} ms</strong></h3>
  <div id="game-canvas-container">
    <canvas id="game-canvas" height="200"/>
  </div>
</template>

<style lang="scss" scoped>
@import "@/scss/custom";

.speed-config-container {
  gap: map-get($spacers, 2);
}
</style>
