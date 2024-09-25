<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import p5 from "p5";

  // Parameters to tweak
  const gravity = 0.01;  // Gravity strength
  const pixelSize = 1;  // Size of each falling pixel
  const generationRate = 100;  // Pixels generated per frame
  const fallSpeedMin = -4;  // Minimum initial speed of falling pixels
  const fallSpeedMax = -1;  // Maximum initial speed of falling pixels
  const initialXSpeedRange = 6; // Range for initial random horizontal velocity

  // Array to hold the falling pixels
  let fallingPixels: Pixel[] = [];
  let canvasContainer: HTMLElement | null = null;
  let sketchInstance: p5 | null = null;

  class Pixel {
    x: number;
    y: number;
    xSpeed: number;
    ySpeed: number;

    constructor(x: number, y: number, xSpeed: number, ySpeed: number) {
      this.x = x;
      this.y = y;
      this.xSpeed = xSpeed;
      this.ySpeed = ySpeed;
    }

    // Update pixel position with gravity
    update() {
      this.ySpeed += gravity;
      this.x += this.xSpeed;
      this.y += this.ySpeed;
    }

    // Render the pixel as a square
    display(p: p5) {
      p.fill(255);
      p.noStroke();
      p.rect(this.x, this.y, pixelSize, pixelSize);
    }

    // Check if pixel is off-screen
    isOffScreen(p: p5) {
      return this.y > p.height || this.x < 0 || this.x > p.width;
    }
  }

  const sketch = (p: p5) => {
    p.setup = () => {
      p.createCanvas(p.windowWidth, p.windowHeight).parent(canvasContainer);
      p.background(0);
      p.angleMode(p.DEGREES);
    };

    p.draw = () => {
      p.background(0);
      const x1 = 0;
      const y1 = p.height * 0.20;
      const x2 = p.width * 0.8;
      const y2 = p.height * 0.85;

      // Generate pixels along the line's path
      for (let i = 0; i < generationRate; i++) {
        //const t = Math.random()*Math.random(); // A random point along the line
        const t = Math.random()
        const x = p.lerp(x2, x1, t);
        const y = p.lerp(y2, y1, t);
        const xSpeed = p.random(-initialXSpeedRange, initialXSpeedRange);
        const ySpeed = p.random(fallSpeedMin, fallSpeedMax);
        fallingPixels.push(new Pixel(x, y, xSpeed, ySpeed));
      }

      // Update and display falling pixels
      for (let i = fallingPixels.length - 1; i >= 0; i--) {
        const pixel = fallingPixels[i];
        pixel.update();
        pixel.display(p);

        // Remove pixels that are off-screen
        if (pixel.isOffScreen(p)) {
          fallingPixels.splice(i, 1);
        }
      }
    };

    // Handle window resizing
    p.windowResized = () => {
      p.resizeCanvas(p.windowWidth, p.windowHeight);
    };
  };

  // Initialize p5 when the component is mounted
  onMount(() => {
    sketchInstance = new p5(sketch);
  });

  // Clean up p5 when the component is destroyed
  onDestroy(() => {
    if (sketchInstance) {
      sketchInstance.remove();
    }
  });
</script>

<!-- Bind the canvas to this div -->
<div bind:this={canvasContainer}></div>

<style>
  div {
    width: 100vw;
    height: 100vh;
    margin: 0;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
  }
</style>
