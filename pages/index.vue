<script setup lang="ts">
import { onMounted } from '#imports'
useHead({ title: "1D Minesweeper" });

let cellSize: number = 10;
let height: number = 20;
let width: number = 20;
let scene, ctx;
const score = ref(0);
let isOver: boolean = false;

const floor = new Image();
floor.src = "./floor.png";
await new Promise((resolve) => {
    floor.onload = () => resolve(1);
});

const foodTexture = new Image();
foodTexture.src = "./food.png";
await new Promise((resolve) => {
    foodTexture.onload = () => resolve(1);
});

const snake = {
    x: 0,
    y: 0,
    velocityX: 0,
    velocityY: 0,
    tail: []
}

const food = {
    x: 0,
    y: 0
}

onMounted(async ()=>{
    scene = document.getElementById("scene");
    scene.height = height * cellSize;
    scene.width = width * cellSize;
    ctx = scene.getContext("2d");

    await updateFoodPos();
    document.addEventListener("keydown", changeDirection);
    setInterval(renderFrame, 66.67); // 1000/15 = 66.67ms - 15fps
});

async function renderFrame() {
    if (isOver) {
        return;
    }

    // background
    // ctx.fillStyle = "black";
    // ctx.fillRect(0, 0, scene.width, scene.height);
    ctx.drawImage(floor, 0, 0);

    // food
    // ctx.fillStyle = 'rgb(219,157,169)';
    // ctx.fillRect(food.x, food.y, cellSize, cellSize);
    ctx.drawImage(foodTexture, food.x, food.y);

    // snake
    if (snake.x === food.x && snake.y === food.y) {
        snake.tail.push([food.x, food.y]);
        updateFoodPos();
    }

    for (let i = snake.tail.length - 1; i > 0; i--) {
        snake.tail[i] = snake.tail[i - 1];
    }

    if (snake.tail.length) {
        snake.tail[0] = [snake.x, snake.y];
    }

    snake.x += snake.velocityX * cellSize;
    snake.y += snake.velocityY * cellSize;

    if (snake.x < 0) {
        snake.x = (width - 1) * cellSize;
    }
    if (snake.x > (width - 1) * cellSize) {
        snake.x = 0;
    }
    if (snake.y < 0) {
        snake.y = (height - 1) * cellSize;
    }
    if (snake.y > (height - 1) * cellSize) {
        snake.y = 0;
    }

    ctx.fillStyle = 'white';
    ctx.fillRect(snake.x, snake.y, cellSize, cellSize);

    // tail
    ctx.fillStyle = 'rgb(237, 230, 204)';
    for (let i = 0; i < snake.tail.length; i++) {
        ctx.fillRect(snake.tail[i][0], snake.tail[i][1], cellSize, cellSize);
    }

    //game over
    for (let i = 0; i < snake.tail.length; i++) {
        if (snake.x === snake.tail[i][0] && snake.y === snake.tail[i][1]) {
            gameOver();
        }
    }

    ditherFrame();
}

async function ditherFrame() {
    const lightRadius = 125;
    const shadingCanvas = document.createElement("canvas");
    shadingCanvas.width = scene.width;
    shadingCanvas.height = scene.height;
    const shadingCtx = shadingCanvas.getContext("2d");
    shadingCtx.fillStyle = "black";
    shadingCtx.fillRect(0, 0, scene.width, scene.height);

    let gradient = shadingCtx.createRadialGradient(snake.x, snake.y, 0, snake.x, snake.y, lightRadius);
    gradient.addColorStop(0.0, "rgba(255, 255, 255, 255)");
    gradient.addColorStop(1.0, "rgba(255, 255, 255, 0)");
    shadingCtx.beginPath();
    shadingCtx.fillStyle = gradient;
    shadingCtx.arc(snake.x, snake.y, lightRadius, 0, 2 * Math.PI);
    shadingCtx.fill();

    let gradient2 = shadingCtx.createRadialGradient(food.x, food.y, 0, food.x, food.y, lightRadius / 2);
    gradient2.addColorStop(0.0, "rgba(255, 255, 255, 255)");
    gradient2.addColorStop(1.0, "rgba(255, 255, 255, 0)");
    shadingCtx.fillStyle = gradient2;
    shadingCtx.arc(food.x, food.y, lightRadius / 2, 0, 2 * Math.PI);
    shadingCtx.fill();

    // screen overlay mode
    let frame = ctx.getImageData(0, 0, scene.width, scene.height);
    let mask = shadingCtx.getImageData(0, 0, shadingCanvas.width, shadingCanvas.height);

    for (let i = 0; i < frame.data.length; i += 4) {
        const brightness = mask.data[i] / 255;
        frame.data[i] *= brightness;
        frame.data[i + 1] *= brightness;
        frame.data[i + 2] *= brightness;
    }

    ctx.putImageData(frame, 0, 0);

    let image = ctx.getImageData(0, 0, scene.width, scene.height);
    image = bayer(image, 128);
    ctx.putImageData(image, 0, 0);
}

function bayer(image, threshold) {
    const thresholdMap = [
      [15, 135, 45, 165],
      [195, 75, 225, 105],
      [60, 180, 30, 150],
      [240, 120, 210, 90],
    ];

    for (let i = 0; i < image.data.length; i += 4) {
      const luminance = (image.data[i] * 0.299) + (image.data[i + 1] * 0.587) + (image.data[i + 2] * 0.114);

      const x = i / 4 % image.width;
      const y = Math.floor(i / 4 / image.width);
      const map = Math.floor((luminance + thresholdMap[x % 4][y % 4]) / 2);
      const value = map < threshold ? 0 : 255;
      image.data.fill(value, i, i + 3);
    }
    return image;
  }

async function updateFoodPos() {
    food.x = Math.floor(Math.random() * width) * cellSize;
    food.y = Math.floor(Math.random() * height) * cellSize;
    score.value++;
}

function changeDirection(e) {
    if ((e.code === "ArrowUp" || e.code === "KeyW") && snake.velocityY != 1) {
        snake.velocityX = 0;
        snake.velocityY = -1;
    }
    else if ((e.code === "ArrowDown" || e.code === "KeyS") && snake.velocityY != -1) {
        snake.velocityX = 0;
        snake.velocityY = 1;
    }
    else if ((e.code === "ArrowLeft" || e.code === "KeyA") && snake.velocityX != 1) {
        snake.velocityX = -1;
        snake.velocityY = 0;
    }
    else if ((e.code === "ArrowRight" || e.code === "KeyD") && snake.velocityX != -1) {
        snake.velocityX = 1;
        snake.velocityY = 0;
    }
}

async function gameOver() {
    isOver = true;
    score.value = 0;
    snake.tail = [];
}
</script>

<template>
    <div class="flex justify-center flex-wrap flex-col items-center min-w-[300px] select-none">
        <p>Score: {{ score }}</p>
        <canvas id="scene" class="w-[95vw]" style="image-rendering: pixelated"></canvas>
    </div>
</template>