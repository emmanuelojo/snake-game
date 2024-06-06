<script setup lang="ts">
const toast = useToast();

const playboard = ref();
// const scoreElement = ref()
// const highScoreElement = ref()
const controls = ref();

const gameOver = ref(false);
const foodX = ref<number | null>(null);
const foodY = ref<number | null>(null);

const snakeX = ref(5);
const snakeY = ref(5);
const velocityX = ref(0);
const velocityY = ref(0);
const snakeBody = ref<any[]>([]);
let setIntervalId: any;
const score = ref(0);

// get highScore from localStorage
const highScore = ref(0);

// pass a random number between 1-30
const notificationActions = [
    {
        label: "Replay",
        click: () => replay(),
    },
];

onMounted(() => {
    playboard.value = document.querySelector(".play-board");
    controls.value = document.querySelectorAll(".controls .icon");
    highScore.value = Number(localStorage.getItem("high-score")) || 0;

    updateFoodPosition();
    beginGameToast();
    setIntervalId = setInterval(initGame, 100);
});

function updateFoodPosition() {
    foodX.value = Math.floor(Math.random() * 30) + 1;
    foodY.value = Math.floor(Math.random() * 30) + 1;

    console.log("food pos: ", foodX.value, foodY.value);
}

function handleGameOver() {
    clearInterval(setIntervalId);
    gameOverToast();
    location.reload();
}

// increase velocity value based on keypress
// change it to be after eating, increase the speed
function changeDirection(direction: string) {
    console.log("direction: ", direction);
    if (direction === "up" && velocityY.value !== 1) {
        velocityX.value = 0;
        velocityY.value = -1;
    } else if (direction === "down" && velocityY.value !== -1) {
        velocityX.value = 0;
        velocityY.value = -1;
    } else if (direction === "left" && velocityX.value !== 1) {
        velocityX.value = -1;
        velocityY.value = 0;
    } else if (direction === "right" && velocityX.value !== -1) {
        velocityX.value = 1;
        velocityY.value = 0;
    }
}

function initGame() {
    if (gameOver.value) return handleGameOver();

    let html = `<div class="food" style="grid-area: ${foodY.value} / ${foodX.value} ;"></div>`;

    // when snake eats food
    if (snakeX.value === foodX.value && snakeY.value === foodY.value) {
        updateFoodPosition();
        snakeBody.value.push([foodY.value, foodX.value]); // add food to snake body array
        score.value++;
        highScore.value = score.value >= highScore.value ? score.value : highScore.value;

        localStorage.setItem("high-score", highScore.value.toString());
    }

    // update snake head
    snakeX.value += velocityX.value;
    snakeY.value += velocityY.value;

    // shifting forward values of elements in snake body by one
    for (let i = snakeBody.value.length - 1; i > 0; i--) {
        snakeBody.value[i] = snakeBody.value[i - 1];
    }

    snakeBody.value[0] = [snakeX.value, snakeY.value];

    // check snake body is out of wall or not
    if (snakeX.value <= 0 || snakeX.value > 30 || snakeY.value <= 0 || snakeY.value > 30) {
        return (gameOver.value = true);
    }

    // add div for each part of snake body
    for (let i = 0; i < snakeBody.value.length; i++) {
        html += `<div class="head" style="grid-area: ${snakeBody.value[i][1]} / ${snakeBody.value[i][0]};"></div>`;

        // check if snake head hit body or not
        if (
            i !== 0 &&
            snakeBody.value[0][1] === snakeBody.value[i][1] &&
            snakeBody.value[0][0] === snakeBody.value[i][0]
        )
            gameOver.value = true;
    }

    playboard.value.innerHTML = html;
}

function replay() {
    location.reload();
}

function beginGameToast() {
    toast.add({ title: "Game is ready.", description: "You can play!!!" });
}

function gameOverToast() {
    toast.add({ title: "Game Over", description: "Press OK to replay...", color: "red", actions: notificationActions });
}
</script>

<template>
    <div class="wrapper">
        <div class="game-details">
            <span class="score">Score: {{ score }}</span>
            <span class="high-score">High Score: {{ highScore }}</span>
        </div>

        <div class="play-board"></div>

        <div class="controls">
            <div @click="changeDirection('left')" class="icon">
                <UIcon data-key="arrow-left" name="i-heroicons-arrow-long-left-solid" />
            </div>
            <div @click="changeDirection('up')" class="icon">
                <UIcon data-key="arrow-up" name="i-heroicons-arrow-long-up-solid" />
            </div>
            <div @click="changeDirection('down')" class="icon">
                <UIcon data-key="arrow-down" name="i-heroicons-arrow-long-down-solid" />
            </div>
            <div @click="changeDirection('right')" class="icon">
                <UIcon data-key="arrow-right" name="i-heroicons-arrow-long-right-solid" />
            </div>
        </div>
    </div>
</template>

<style scoped>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: "Open Sans", sans-serif;
}

body {
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    background: #e3f2fd;
}

.wrapper {
    width: 65vmin;
    height: 70vmin;
    display: flex;
    overflow: hidden;
    flex-direction: column;
    justify-content: center;
    background: #293447;
    border-radius: 5px;
    box-shadow: 0 20px 40px rgba(52, 87, 220, 0.2);
}

.game-details {
    color: #b8c6dc;
    font-weight: 500;
    font-size: 1.2rem;
    padding: 20px 27px;
    display: flex;
    justify-content: space-between;
}

.play-board {
    height: 100%;
    width: 100%;
    display: grid;
    background: #212837;
    grid-template: repeat(30, 1fr) / repeat(30, 1fr);
}

.play-board .food {
    background: #ff003d;
    border-radius: 50%;
}

.play-board .head {
    background: #60cbff;
}

.controls {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.controls .icon {
    padding: 25px 0;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.3rem;
    color: #b8c6dc;
    width: calc(100% / 4);
    cursor: pointer;
    border-right: 1px solid #171b26;
}

@media screen and (max-width: 800px) {
    .wrapper {
        width: 90vmin;
        height: 115vmin;
    }

    .game-details {
        font-size: 1rem;
        padding: 15px 27px;
    }

    .controls {
        display: flex;
    }

    .controls .icon {
        padding: 15px 0;
        font-size: 1rem;
    }
}
</style>
