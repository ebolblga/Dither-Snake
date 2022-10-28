<script setup lang="ts">
import { onMounted } from '#imports'
useHead({ title: "DitherSnake" });

const cellSize: number = 16;
let width: number = Math.floor(window.innerWidth / 64);
let height: number = Math.floor(window.innerHeight / 64);
let scene, ctx;
const score = ref(0);
const isOver = ref(false);
const fps = 10;

const floor = new Image();
floor.src = "./assets/floor.png";
await new Promise((resolve) => {
    floor.onload = () => resolve(1);
});

const foodTexture = new Image();
foodTexture.src = "./assets/food_atlas.png";
await new Promise((resolve) => {
    foodTexture.onload = () => resolve(1);
});

const particleSprite = new Image();
particleSprite.src = "./assets/particle_sprite.png";
await new Promise((resolve) => {
    particleSprite.onload = () => resolve(1);
});

const foodSound = new Audio("./sfx/food.wav");
foodSound.volume = 0.1;

const deathSound = new Audio("./sfx/death.wav");
deathSound.volume = 0.3;

const buffer = {
    velocityX: 0,
    velocityY: 0
}

const snake = {
    x: 0,
    y: 0,
    velocityX: 0,
    velocityY: 0,
    tail: [],
    maxTail: 1
}

const food = {
    index: 0,
    x: 1,
    y: 1,
    particleFrame: 0,
}

onMounted(async ()=>{
    scene = document.getElementById("scene");
    scene.height = height * cellSize;
    scene.width = width * cellSize;
    ctx = scene.getContext("2d");

    await updateFoodPos();
    document.addEventListener("keydown", changeDirection);
    window.addEventListener("resize", updateRes);
    setInterval(function () {
        renderFrame();
    }, 1000 / fps);  // 1000 / 15 = 66.67ms - 15fps
});

function updateRes() {
    width = Math.floor(window.innerWidth / 64);
    height = Math.floor(window.innerHeight / 64);

    scene.height = height * cellSize;
    scene.width = width * cellSize;

    if (food.x > width - 1) {
        updateFoodPos();
    }
    if (food.y > height - 1) {
        updateFoodPos();
    }
    
    renderFrame();
}

async function renderFrame() {
    if (isOver.value) {
        return;
    }

    // background
    ctx.drawImage(floor, 0, 0, scene.width, scene.height);

    // food
    ctx.drawImage(foodTexture, food.index * cellSize, 0, cellSize, cellSize, food.x * cellSize, food.y * cellSize, cellSize, cellSize);

    snake.velocityX = buffer.velocityX;
    snake.velocityY = buffer.velocityY;

    snake.x += snake.velocityX;
    snake.y += snake.velocityY;

    if (snake.x < 0) {
        snake.x = (width - 1);
    }
    else if (snake.x > (width - 1)) {
        snake.x = 0;
    }

    if (snake.y < 0) {
        snake.y = (height - 1);
    }
    else if (snake.y > (height - 1)) {
        snake.y = 0;
    }

    snake.tail.unshift({ x: snake.x, y: snake.y });

    if (snake.tail.length > snake.maxTail) {
		snake.tail.pop();
	}
    
    ctx.fillStyle = "white";
    snake.tail.forEach( function(cell, index) {
        ctx.fillRect(cell.x * cellSize, cell.y * cellSize, cellSize, cellSize);

        if (cell.x === food.x && cell.y === food.y) {
            foodSound.play();
            snake.maxTail++;
            score.value++;
            snake.tail.push([food.x, food.y]);
            updateFoodPos();
        }

        for( let i = index + 1; i < snake.tail.length; i++ ) {
            if ( cell.x == snake.tail[i].x && cell.y == snake.tail[i].y ) {
                deathSound.play();
                isOver.value = true;
            }
        }
    });

    ditherFrame();
}

async function ditherFrame() {
    //let lightRadius = 125;
    const lightRadius = Math.floor(Math.random() * (130 - 120) + 120);

    const shadingCanvas = document.createElement("canvas");
    shadingCanvas.width = scene.width;
    shadingCanvas.height = scene.height;
    const shadingCtx = shadingCanvas.getContext("2d");
    shadingCtx.fillStyle = "black";
    shadingCtx.fillRect(0, 0, scene.width, scene.height);

    const snakeX = snake.x * cellSize + (cellSize / 2);
    const snakeY = snake.y * cellSize + (cellSize / 2);
    let gradient = shadingCtx.createRadialGradient(snakeX, snakeY, 0, snakeX, snakeY, lightRadius);
    gradient.addColorStop(0.0, "rgba(255, 255, 255, 255)");
    gradient.addColorStop(1.0, "rgba(255, 255, 255, 0)");
    shadingCtx.beginPath();
    shadingCtx.fillStyle = gradient;
    shadingCtx.arc(snakeX, snakeY, lightRadius, 0, 2 * Math.PI);
    shadingCtx.fill();

    const foodX = food.x * cellSize + (cellSize / 2);
    const foodY = food.y * cellSize + (cellSize / 2);
    let gradient2 = shadingCtx.createRadialGradient(foodX, foodY, 10, foodX, foodY, lightRadius / 3);
    gradient2.addColorStop(0.0, "rgba(255, 255, 255, 255)");
    gradient2.addColorStop(1.0, "rgba(255, 255, 255, 0)");
    shadingCtx.fillStyle = gradient2;
    shadingCtx.arc(foodX, foodY, lightRadius / 3, 0, 2 * Math.PI);
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

    //particles
    food.particleFrame++;
    if (food.particleFrame > 25) {
        food.particleFrame = 0;
    }

    ctx.drawImage(particleSprite, food.particleFrame * cellSize, 0, cellSize, cellSize * 2, food.x * cellSize, (food.y - 3) * cellSize, cellSize, cellSize * 2);
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
    let isPlaced = false;
    while (!isPlaced) {
        food.x = Math.floor(Math.random() * width);
        food.y = Math.floor(Math.random() * height);

        isPlaced = true;
        snake.tail.forEach( function(cell) {
            if (cell.x === food.x && cell.y === food.y) {
                isPlaced = false;
            }
        });
    }

    food.index = Math.floor(Math.random() * 5);
}

function changeDirection(e) {
    if ((e.code === "ArrowUp" || e.code === "KeyW") && snake.velocityY != 1) {
        buffer.velocityX = 0;
        buffer.velocityY = -1;
    }
    else if ((e.code === "ArrowDown" || e.code === "KeyS") && snake.velocityY != -1) {
        buffer.velocityX = 0;
        buffer.velocityY = 1;
    }
    else if ((e.code === "ArrowLeft" || e.code === "KeyA") && snake.velocityX != 1) {
        buffer.velocityX = -1;
        buffer.velocityY = 0;
    }
    else if ((e.code === "ArrowRight" || e.code === "KeyD") && snake.velocityX != -1) {
        buffer.velocityX = 1;
        buffer.velocityY = 0;
    }
    
    if (isOver.value && e.code === "KeyX") {
        resetGame();
    }
}

async function resetGame() {
    score.value = 0;
    isOver.value = false;
    snake.x = snake.y = snake.velocityX = snake.velocityY = 0;
    snake.maxTail = 1;
    snake.tail = [];
    buffer.velocityX = buffer.velocityY = 0;
    updateFoodPos();
}
</script>

<template>
    <div class="h-full select-none">
        <canvas id="scene" class="z-0 w-full h-full" style="image-rendering: pixelated"></canvas>
        <p class="fixed top-5 right-5 bg-black">Score: {{ score }}</p>
        <div v-if="isOver">
            <p class="fixed top-10 right-5 text-lg bg-black">Game over</p>
            <p class="fixed top-16 right-5 text-xs bg-black">Press X to restart</p>
        </div>
    </div>
</template>