<script lang="ts">
	import { onMount } from "svelte";

  let Canvas: HTMLCanvasElement;
  // let LayerOne: HTMLCanvasElement;
  // let LayerTwo: HTMLCanvasElement;
  let context: CanvasRenderingContext2D;

  let ColorPicker: HTMLInputElement;

  type Tool = "pencil" | "eraser";

  let coords = $state({
    x: 0,
    y: 0
  });
  let isMouseDown: boolean = $state(false);
  let currentTool: Tool = $state("pencil");
  // Pencil Settings
  let activeColor: string = $state("#000000");
  let pencilSize: number = $state(10);
  // Eraser Settings
  let eraserSize: number = $state(10);

  function draw(event: MouseEvent) {
    if (event.target !== Canvas) {
      return;
    }

    if (isMouseDown) {
      if (currentTool === "pencil") {
        context.globalCompositeOperation = "source-over";
        context.beginPath();
        context.lineWidth = pencilSize;
        context.lineCap = 'round';
        context.strokeStyle = activeColor;
        context.moveTo(coords.x, coords.y);
        reposition(event);
        context.lineTo(coords.x, coords.y);
        context.stroke();
      } else if (currentTool === "eraser") {
        context.globalCompositeOperation = "destination-out";
        context.beginPath();
        context.lineWidth = eraserSize;
        context.lineCap = 'round';
        context.moveTo(coords.x, coords.y);
        reposition(event);
        context.lineTo(coords.x, coords.y);
        context.stroke();
      }
    }
  }

  function reposition(ev: MouseEvent) {
    coords.x = ev.clientX - Canvas.offsetLeft;
    coords.y = ev.clientY - Canvas.offsetTop;
  }

  function start(ev: MouseEvent) {
    // console.log("Mouse is down!");
    isMouseDown = true;
    reposition(ev);
  }

  function stop() {
    // console.log("Mouse is up!");
    isMouseDown = false;
  }

  function setTool(tool: Tool) {
    currentTool = tool;
  }

  function clearCanvas() {
    if (Canvas) {
      context.clearRect(0, 0, Canvas.width, Canvas.height);
    }
  }

  // $inspect(coords);

  onMount(() => {
    if (Canvas) {
      Canvas.width = window.innerWidth;
      Canvas.height = window.innerHeight;
      // @ts-expect-error - Type 'CanvasRenderingContext2D | null' is not assignable to type 'CanvasRenderingContext2D'
      context = Canvas.getContext("2d");
    }
  });
</script>

<svelte:document on:mousedown={start} on:mouseup={stop} on:mousemove={draw} />

<div class="app">
  <div class="controls">
    <div class="container">
      <button class="btn pencil" aria-label="Pencil"
        onclick={() => { setTool('pencil'); }} disabled={currentTool === 'pencil'}>
        <span aria-hidden="true">Pencil</span>
      </button>
      <button class="btn eraser" aria-label="Eraser"
        onclick={() => { setTool('eraser'); }} disabled={currentTool === 'eraser'}>
        <span aria-hidden="true">Eraser</span>
      </button>
      <button class="btn clear-canvas" aria-label="Clear canvas" onclick={clearCanvas}>
        <span aria-hidden="true">Clear Canvas</span>
      </button>
      <button class="btn export" aria-label="Export drawing" onclick={exportCanvas}>
        <span aria-hidden="true">Export</span>
      </button>
    </div>
    <div class="settings">
      {#if currentTool === "pencil"}
        <div class="pencil-settings">
          <button
            class="btn color-picker" aria-label="Current color: {activeColor}"
            onclick={() => { ColorPicker.click(); }}>
            <div class="swatch-viewport" style="background-color: {activeColor}" aria-hidden="true"></div>
            <span aria-hidden="true">{activeColor}</span>
          </button>
          <input type="color" tabindex="-1" bind:value={activeColor} bind:this={ColorPicker} />
          <input type="number" min="10" max="400" step="10" bind:value={pencilSize} />
          <input type="range" min="10" max="400" step="10" bind:value={pencilSize} />
        </div>
      {:else if currentTool === "eraser"}
        <div class="eraser-settings">
          <input type="number" min="10" max="400" step="10" bind:value={eraserSize} />
          <input type="range" min="10" max="400" step="10" bind:value={eraserSize} />
        </div>
      {/if}
    </div>
  </div>
  <canvas bind:this={Canvas} width="600" height="600"></canvas>
</div>

<style>
  div.app {
    width: 100dvw;
    height: 100dvh;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    gap: 40px;
  }

  canvas {
    width: 100%;
    height: 100%;
    cursor: crosshair;
  }

  .controls {
    width: 100%;
    display: flex;
    justify-content: flex-start;
    align-items: flex-start;
    padding: 20px 20px 0px;
    position: fixed;
    top: 0;
    inset-inline: 0;
    pointer-events: none;

    & > div.container {
      width: max-content;
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      gap: 10px;
      padding: 10px;
      border-radius: 9999rem;
      border: 2px solid color-mix(in oklab, black, transparent 80%);
      background-color: white;
      pointer-events: all;
    }

    & > div.settings {
      width: max-content;
      position: absolute;
      top: calc(100% + 10px);
      padding: 10px;
      border-radius: 9999rem;
      border: 2px solid color-mix(in oklab, black, transparent 80%);
      background: white;
      pointer-events: all;
    }
  }

  button.btn {
    color: var(--clr-onyx);
    padding-inline: 20px;
    padding-block: 10px;

    &:is(:hover, :focus) {
      color: var(--clr-night);
      background-color: var(--clr-platinum);
    }

    &:disabled {
      color: var(--clr-white);
      background-color: var(--clr-tangelo);
    }
  }

  button.btn.color-picker {
    gap: 5px;
    padding-inline: 10px 20px;

    & > div.swatch-viewport {
      width: 20px;
      height: 20px;
      aspect-ratio: 1 / 1;
      border-radius: 9999rem;
    }
  }

  div.pencil-settings {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    gap: 10px;

    & > input[type="color"] {
      width: 1px;
      height: 1px;
      margin: -1px;
      clip-path: rect(0,0,0,0);
      pointer-events: none;
      opacity: 0;
      position: absolute;
    }
    & > input[type="number"] {
      padding: 10px;
      border-radius: 9999rem;
      border: 2px solid color-mix(in oklab, black, transparent 80%);
      background-color: transparent;
      outline: 2px solid transparent;
      outline-offset: 2px;

      &:is(:hover, :focus) {
        border-color: color-mix(in oklab, black, transparent 50%);
      }
      &:focus-visible {
        outline-color: var(--clr-blue);
      }
    }
    & > input[type="range"] {
      padding: 5px;
      border-radius: 9999rem;
      outline: 2px solid transparent;
      outline-offset: 2px;

      &:focus-visible {
        outline-color: var(--clr-blue);
      }
    }
  }

  div.eraser-settings {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    gap: 10px;
    
    & > input[type="number"] {
      padding: 10px;
      border-radius: 9999rem;
      border: 2px solid color-mix(in oklab, black, transparent 80%);
      background-color: transparent;
      outline: 2px solid transparent;
      outline-offset: 2px;

      &:is(:hover, :focus) {
        border-color: color-mix(in oklab, black, transparent 50%);
      }
      &:focus-visible {
        outline-color: var(--clr-blue);
      }
    }
    & > input[type="range"] {
      padding: 5px;
      border-radius: 9999rem;
      outline: 2px solid transparent;
      outline-offset: 2px;

      &:focus-visible {
        outline-color: var(--clr-blue);
      }
    }
  }
</style>
