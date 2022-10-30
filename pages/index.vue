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
const bgColor = ref("rgb(0, 0, 0)");
const textColor = ref("rgb(255, 255, 255)");

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

const buffer = {
    velocityX: 0,
    velocityY: 0
}

const touch = {
    startX: 0,
    startY: 0,
    endX: 0,
    endY: 0
}

const dither = {
    index: 0,
    light: [
        [255, 255, 255],    // #FFFFFF
        [244, 78, 56],      // #F44E38
        [199, 199, 199],    // #C7C7C7           
        [195, 220, 229],    // #C3DCE5
        [237, 246, 214],    // #EDF6D6
        [235, 229, 206],    // #EBE5CE
        [114, 164, 136],    // #72A488
        [34, 35, 35]       // #222324
    ],

    dark: [
        [0, 0, 0],          // #000000
        [29, 15, 68],       // #1D0F44
        [20, 23, 36],       // #141724    
        [17, 12, 34],       // #110C22
        [62, 35, 44],       // #3E232C
        [46, 48, 55],       // #2E3037
        [33, 44, 40],       // #212C28
        [240, 246, 240]    // #F0F6F0
    ]
}

const floor = new Image();
floor.src = "./assets/floor_texture.png";
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

document.addEventListener("touchstart", e => {
    touch.startX = e.touches[0].clientX;
    touch.startY = e.touches[0].clientY;

    touch.endX = e.touches[0].clientX;
    touch.endY = e.touches[0].clientY;
});

document.addEventListener("touchmove", e => {
    touch.endX = e.touches[0].clientX;
    touch.endY = e.touches[0].clientY;
});

document.addEventListener("touchend", e => {
    let differenceX = touch.startX - touch.endX;
    let differenceY = touch.startY - touch.endY;

    if ((Math.abs(differenceY) + Math.abs(differenceX)) < 20) {
        if (isOver.value) {
            resetGame();
        }

        return;
    }

    if (Math.abs(differenceY) > Math.abs(differenceX)) {
        if (differenceY > 0 && snake.velocityY != 1) {
            buffer.velocityX = 0;
            buffer.velocityY = -1;
        }
        else if (differenceY <= 0 && snake.velocityY != -1) {
            buffer.velocityX = 0;
            buffer.velocityY = 1;
        }
    }
    else if (Math.abs(differenceY) < Math.abs(differenceX)) {
        if (differenceX > 0 && snake.velocityX != 1) {
            buffer.velocityX = -1;
            buffer.velocityY = 0;
        }
        else if (differenceX <= 0 && snake.velocityX != -1) {
            buffer.velocityX = 1;
            buffer.velocityY = 0;
        }
    }
});

onMounted(async ()=>{
    scene = document.getElementById("scene");
    scene.height = height * cellSize;
    scene.width = width * cellSize;
    ctx = scene.getContext("2d");

    await updateFoodPos();
    document.addEventListener("keydown", eventHandler);
    window.addEventListener("resize", updateRes);
    setInterval(function () {
        renderFrame();
    }, 1000 / fps);  // 1000 / 15 = 66.67ms - 15fps
});

async function renderFrame() {
    if (isOver.value) {
        return;
    }
    const start = Date.now();

    // background
    ctx.drawImage(floor, 0, 0, scene.width, scene.height);

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

    shadeFrame();
    //remove await
    // console.log(Date.now() - start + "ms");
}

async function shadeFrame() {
    const lightR = 165;
    const flicker = 5;
    const lightRadius = Math.floor(Math.random() * ((lightR + flicker) - (lightR - flicker)) + (lightR - flicker));

    const shadingCanvas = document.createElement("canvas");
    shadingCanvas.width = scene.width;
    shadingCanvas.height = scene.height;
    const shadingCtx = shadingCanvas.getContext("2d");
    shadingCtx.fillStyle = "black";
    shadingCtx.fillRect(0, 0, scene.width, scene.height);

    const snakeX = snake.x * cellSize + (cellSize / 2);
    const snakeY = snake.y * cellSize + (cellSize / 2);
    let gradient = shadingCtx.createRadialGradient(snakeX, snakeY, 0, snakeX, snakeY, lightRadius);
    // gradient.addColorStop(0.0, "rgba(255, 255, 255, 255)");
    // gradient.addColorStop(1.0, "rgba(255, 255, 255, 0)");

    // realistic light falloff
    gradient.addColorStop(0.0, "rgba(255, 255, 255, 255)");
    gradient.addColorStop(0.1, "rgba(206, 206, 206, 206)");
    gradient.addColorStop(0.2, "rgba(163, 163, 163, 163)");
    gradient.addColorStop(0.3, "rgba(125, 125, 125, 125)");
    gradient.addColorStop(0.4, "rgba(92, 92, 92, 92)");
    gradient.addColorStop(0.5, "rgba(64, 64, 64, 64)");
    gradient.addColorStop(0.6, "rgba(41, 41, 41, 41)");
    gradient.addColorStop(0.7, "rgba(23, 23, 23, 23)");
    gradient.addColorStop(0.8, "rgba(10, 10, 10, 10)");
    gradient.addColorStop(0.9, "rgba(2, 2, 2, 2)");
    gradient.addColorStop(1.0, "rgba(0, 0, 0, 0)");

    shadingCtx.beginPath();
    shadingCtx.fillStyle = gradient;
    shadingCtx.arc(snakeX, snakeY, lightRadius, 0, 2 * Math.PI);
    shadingCtx.fill();

    const foodX = food.x * cellSize + (cellSize / 2);
    const foodY = food.y * cellSize + (cellSize / 2);
    let gradient2 = shadingCtx.createRadialGradient(foodX, foodY, 8, foodX, foodY, lightRadius / 4);
    gradient2.addColorStop(0.0, "rgba(255, 255, 255, 255)");
    gradient2.addColorStop(1.0, "rgba(255, 255, 255, 0)");
    shadingCtx.fillStyle = gradient2;
    shadingCtx.arc(foodX, foodY, lightRadius / 3, 0, 2 * Math.PI);
    shadingCtx.fill();

    // shadow
    let shadowMask = foodShadow(snakeX, snakeY, foodX, foodY, lightR + flicker);

    // screen overlay mode
    let lightMask = shadingCtx.getImageData(0, 0, shadingCanvas.width, shadingCanvas.height);
    for (let i = 0; i < lightMask.data.length; i += 4) {
        const brightness = shadowMask.data[i] / 255;
        lightMask.data[i] *= brightness;
        // lightMask.data[i + 1] *= brightness;
        // lightMask.data[i + 2] *= brightness;
    }

    let frame = ctx.getImageData(0, 0, scene.width, scene.height);
    for (let i = 0; i < frame.data.length; i += 4) {
        const brightness = lightMask.data[i] / 255;
        frame.data[i] *= brightness;
        frame.data[i + 1] *= brightness;
        frame.data[i + 2] *= brightness;
    }

    ctx.putImageData(frame, 0, 0);

    // food
    ctx.drawImage(foodTexture, food.index * cellSize, 0, cellSize, cellSize, food.x * cellSize, food.y * cellSize, cellSize, cellSize);

    // dithering
    let image = ctx.getImageData(0, 0, scene.width, scene.height);
    image = bayer4x4(image, 128);
    ctx.putImageData(image, 0, 0);

    //particles
    food.particleFrame++;
    if (food.particleFrame > 25) {
        food.particleFrame = 0;
    }

    ctx.drawImage(particleSprite, food.particleFrame * cellSize, 0, cellSize, cellSize * 2, food.x * cellSize, (food.y - 3) * cellSize, cellSize, cellSize * 2);

    // light on edges
    const radius = Math.floor(lightRadius / 4);
    ctx.strokeStyle = textColor.value;
    ctx.lineWidth = 2;
    if (foodX + radius > scene.width) {
        let leftOver = foodX + radius - scene.width;
        ctx.beginPath();
        ctx.moveTo(scene.width, foodY - leftOver);
        ctx.lineTo(scene.width, foodY + leftOver);
        ctx.stroke();
    }
    
    if (foodX - radius < 0) {
        let leftOver = foodX - radius;
        ctx.beginPath();
        ctx.moveTo(0, foodY - leftOver);
        ctx.lineTo(0, foodY + leftOver);
        ctx.stroke();
    }

    if (foodY + radius > scene.height) {
        let leftOver = foodY + radius - scene.height;
        ctx.beginPath();
        ctx.moveTo(foodX - leftOver, scene.height);
        ctx.lineTo(foodX + leftOver, scene.height);
        ctx.stroke();
    }

    if (foodY - radius < 0) {
        let leftOver = foodY - radius;
        ctx.beginPath();
        ctx.moveTo(foodX - leftOver, 0);
        ctx.lineTo(foodX + leftOver, 0);
        ctx.stroke();
    }
}

function foodShadow(snakeX, snakeY, foodX, foodY, radius) {
    // finds 2 tangent points to circle around food from snake
    const points = tangentPoints(snakeX, snakeY, foodX, foodY, 8);

    // extends tangent lines by the radius of light
    const dx1 = points[0].x - snakeX;
    const dy1 = points[0].y - snakeY;
    const hypot1 = Math.hypot(dx1, dy1)
    const newPoint1X = points[1].x + ((dx1 / hypot1) * radius);
    const newPoint1Y = points[1].y + ((dy1 / hypot1) * radius);

    const dx2 = points[1].x - snakeX;
    const dy2 = points[1].y - snakeY;
    const hypot2 = Math.hypot(dx2, dy2)
    const newPoint2X = points[1].x + ((dx2 / hypot2) * radius);
    const newPoint2Y = points[1].y + ((dy2 / hypot2) * radius);

    // draws shadow canvas
    const shadowCanvas = document.createElement("canvas");
    shadowCanvas.width = scene.width;
    shadowCanvas.height = scene.height;
    const shadowCtx = shadowCanvas.getContext("2d");
    shadowCtx.fillStyle = "#FFFFFF";
    shadowCtx.fillRect(0, 0, scene.width, scene.height);

    shadowCtx.beginPath();
    shadowCtx.moveTo(points[0].x, points[0].y);
    shadowCtx.lineTo(newPoint1X, newPoint1Y);
    shadowCtx.lineTo(newPoint2X, newPoint2Y);
    shadowCtx.lineTo(points[1].x, points[1].y);
    shadowCtx.closePath();

    const num = ((hypot1 / (radius / 2)) - 1) * 255;
    const color = Math.min(Math.max(num, 100), 255);
    const grade = "rgb(" + color + ", "
                         + color + ", "
                         + color + ")";

    shadowCtx.filter = 'blur(4px)';
    shadowCtx.fillStyle = grade;
    shadowCtx.fill();

    return shadowCtx.getImageData(0, 0, shadowCanvas.width, shadowCanvas.height);
}

function tangentPoints(px, py, cx, cy, radius) {
    const dx = cx - px;
    const dy = cy - py;
    const dd = Math.sqrt(dx * dx + dy * dy);
    const a = Math.asin(radius / dd);
    const b = Math.atan2(dy, dx);

    // console.log("dx = " + dx);
    // console.log("dy = " + dy);
    // console.log("dd = " + dd);
    // console.log("a = " + a);
    // console.log("b = " + b);

    let t = b - a
    const ta = {
        x: cx + (radius * Math.sin(t)),
        y: cy + (radius * -Math.cos(t))
    };

    // console.log("t = " + t);
    // console.log("point1X = " + ta.x);
    // console.log("point1Y = " + ta.y);

    t = b + a
    const tb = {
        x: cx + (radius * -Math.sin(t)),
        y: cy + (radius * Math.cos(t))
    };

    // console.log("t = " + t);
    // console.log("point2X = " + tb.x);
    // console.log("point2Y = " + tb.y);

    return [ta, tb];
}

function bayer4x4(image, threshold) {
    const thresholdMap = [
      [15, 135, 45, 165],
      [195, 75, 225, 105],
      [60, 180, 30, 150],
      [240, 120, 210, 90]
    ];

    for (let i = 0; i < image.data.length; i += 4) {
      const luminance = (image.data[i] * 0.299) + (image.data[i + 1] * 0.587) + (image.data[i + 2] * 0.114);

      const x = i / 4 % image.width;
      const y = Math.floor(i / 4 / image.width);
      const map = Math.floor((luminance + thresholdMap[x % 4][y % 4]) / 2);
    //   const value = map < threshold ? 0 : 255;
    //   image.data.fill(value, i, i + 3);

      if (map >= threshold) {
        image.data[i] = dither.light[dither.index][0];
        image.data[i + 1] = dither.light[dither.index][1];
        image.data[i + 2] = dither.light[dither.index][2];
      }
      else {
        image.data[i] = dither.dark[dither.index][0];
        image.data[i + 1] = dither.dark[dither.index][1];
        image.data[i + 2] = dither.dark[dither.index][2];
      }
      
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

function eventHandler(e) {
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

    if (e.code === "KeyC") {
        dither.index++;
        if (dither.index >= dither.light.length) {
            dither.index = 0;
        }

        bgColor.value = "rgb(" + dither.dark[dither.index][0] + ", "
                                + dither.dark[dither.index][1] + ", "
                                + dither.dark[dither.index][2] + ")";

        textColor.value = "rgb(" + dither.light[dither.index][0] + ", "
                                + dither.light[dither.index][1] + ", "
                                + dither.light[dither.index][2] + ")";
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
</script>

<template>
    <div class="h-full select-none">
        <canvas id="scene" class="z-0 w-full h-full" style="image-rendering: pixelated"></canvas>
        <p class="fixed top-5 right-5 bg text">SCORE - {{ score }}</p>
        <div v-if="isOver">
            <p class="fixed top-10 right-5 text-lg bg text">GAME OVER</p>
            <p class="fixed top-16 right-5 text-xs bg text">Press X to restart</p>
        </div>
    </div>
</template>

<style>
.bg {
    background-color: v-bind(bgColor);
}

.text {
    color: v-bind(textColor);
}

body {
    background-color: v-bind(bgColor);
}
</style>