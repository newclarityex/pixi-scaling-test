<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue';
import { Application, Assets, BitmapText, Container, Graphics, ObservablePoint, Point, Sprite } from 'pixi.js';
import { useElementSize } from '@vueuse/core';

const pixiContainer = ref<HTMLDivElement | null>(null);
const pixiCanvas = ref<HTMLCanvasElement | null>(null);
const scaledContainer = ref<Container | null>(null);

const WIDTH = 500;
const HEIGHT = 500;

const pixiApp = ref<Application | null>(null);

const { width, height } = useElementSize(pixiContainer);
const scale = computed(() => {
    const widthScale = width.value / WIDTH;
    const heightScale = height.value / HEIGHT;

    return Math.min(widthScale, heightScale);
});

function resizeRenderer() {
    if (!pixiApp.value || !scaledContainer.value) {
        console.log("NO PIXI APP")
        return;
    };

    // always logs number
    console.log(scale.value)

    // works
    // scaledContainer.value.scale = new Point(scale.value, scale.value) as ObservablePoint;
    
    // causes error and semi works
    scaledContainer.value.scale.set(scale.value, scale.value);

    pixiApp.value.renderer.resolution = 1 / scale.value;
    pixiApp.value.renderer.resize(width.value, height.value, scale.value);

    pixiApp.value.render();
}

watch([width, height], resizeRenderer);

onMounted(async () => {
  pixiApp.value = new Application();
  await pixiApp.value.init({
    canvas: pixiCanvas.value!,
    resolution: window.devicePixelRatio || 1,
    autoDensity: true,
    antialias: true,
    resizeTo: pixiContainer.value!,
  });

  scaledContainer.value = new Container();
  scaledContainer.value.sortableChildren = true;
  pixiApp.value.stage.addChild(scaledContainer.value as Container);

  const renderer = pixiApp.value.renderer;
  const hitboxCircle = new Graphics()
    .circle(0, 0, 50)
    .fill({ color: 'rgba(255, 255, 255, 1)' });
  const hitboxTexture = renderer.generateTexture(hitboxCircle);
  const hitboxSprite = new Sprite(hitboxTexture);
  hitboxSprite.zIndex = 2;
  hitboxSprite.anchor = { x: 0.5, y: 0.5 };
  scaledContainer.value.addChild(hitboxSprite);

  resizeRenderer();
});
</script>

<template>
  <div ref="pixiContainer" class="pixi-container">
    <canvas ref="pixiCanvas" class="pixi-canvas"></canvas>
  </div>
</template>

<style>
html, body, #app {
  height: 100%;
  width: 100%;
}
.pixi-container {
  position: relative;
  border: 2px white solid;
  border-radius: 5px;
  overflow: hidden;
  background-color: black;
  aspect-ratio: 1 / 1;
  height: 100%;
}
</style>