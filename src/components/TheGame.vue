<script setup>
    import { ref } from 'vue';

    const gameState = ref(getStartingGameState());
    const msDelay = ref(200);
    const btnText = ref("START");

    const height = 20;
    const width = 20;

    let interval = null;

    function switchGameState() {
        if (gameState.value.isStarted) {
            btnText.value = "START";
            gameState.value.isStarted = !gameState.value.isStarted;
            interval = clearInterval(interval);
        } else {
            btnText.value = "STOP";
            gameState.value.isStarted = !gameState.value.isStarted;
            interval = setInterval(runCycle, msDelay.value);
        }

    }

    function restartGameState() {
        if (gameState.value.isStarted) {
            switchGameState();
        }
        gameState.value = getStartingGameState();
    }

    function getStartingGameState() {
        const state = {
            isStarted: false,
            board: []
        }

        state.board = genEmptyBoard();
        return state;
    }

    function genEmptyBoard() {
        let emptyBoard = [];
        for (let i=0; i<height; i++) {
            emptyBoard.push(genEmptyRow());
        }
        return emptyBoard;
    }

    function genEmptyRow() {
        let row = [];
        for (let i=0; i<width; i++) {
            row.push(genEmptyBox());
        }
        return row;
    }

    function genEmptyBox() {
        return {
            isAlive: false
        }
    }
    
    function setBoxAlive(box) {
        if (!gameState.value.isStarted) {
            box.isAlive = !box.isAlive;
        }
    }
    
    function runCycle() {
        let newBoard = copyBoard();
        for (let y = 0; y < height; y++) {
            for (let x = 0; x < width; x++) {
                newBoard[y][x].isAlive = checkIfShouldBeAlive(x, y);
            }
        }
        gameState.value.board = newBoard;
    }

    function checkIfShouldBeAlive(x, y) {

        let sum = 0;
        for (let yOffset = -1; yOffset < 2; yOffset++) {
            for (let xOffset = -1; xOffset < 2; xOffset++) {
                if (xOffset == 0 && yOffset == 0) {
                    // skip
                } else {
                    if (checkIsAlive(x + xOffset, y + yOffset) == true) {
                        sum = sum + 1;
                        console.log(x+ " "+ y+" is sum of"+ sum);
                    }
                }
            }
        }   

        if (sum < 2 || sum > 3) {
            return false;
        } else if (sum == 3 && !gameState.value.board[y][x].isAlive) {
            return true;
        } else if (sum >= 2 && sum <= 3 && gameState.value.board[y][x].isAlive == true) {
            return true;
        }
    }

    function checkIsAlive(x, y) {
        let maxWidthIndex = width - 1;
        let maxHeightIndex = height - 1;
        if (x < 0 || x > maxWidthIndex || y < 0 || y > maxHeightIndex) {
            return false;
        } else {
            return gameState.value.board[y][x].isAlive;
        }

    }

    function copyBoard() {
        let newBoard = [];
        for (let y = 0; y<height; y++) {
            let row = [];
            for (let x = 0; x<width; x++) {
                const value = gameState.value.board[y][x];
                row.push({
                    isAlive: value
                })
            }
            newBoard.push(row);
        }
        return newBoard;
    }

    function patternizeInput() {
        msDelay.value = msDelay.value.replace(/\D/g, '');
        if (msDelay.value.length < 1 || msDelay.value < 1) {
            msDelay.value = 1;
        }
    }

</script>

<template> 
    <div class="board">
        <div class="buttons">
            <div class="empty-space"></div>
            <button class="btn" @click="restartGameState">
                RESTART
            </button>
            <button class="btn" @click="switchGameState">
                {{ btnText }}
            </button>
            <input class="btn input-delay" type="text" v-model="msDelay" @input="patternizeInput" :disabled="gameState.isStarted" :class="{'btn-disabled':gameState.isStarted}">
            <button class="btn btn-next" @click="runCycle" :disabled="gameState.isStarted" :class="{'btn-disabled':gameState.isStarted}">
                NEXT CYCLE
            </button>
            <div class="empty-space"></div>
        </div>
        <div class="row" v-for="row in gameState.board">
            <div class="box" 
            v-for="box in row" 
            @click="setBoxAlive(box)"
            :class="{'box-alive': box.isAlive}">
 
            </div>
        </div>
    </div>
</template>

<style>
    .buttons {
        display: flex;
        gap: 5px;
    }

    .btn {
        line-height: 2em;
        font-weight: 600;
        background-color: white;
        color: black;
        border: 1px solid white;
        border-radius: 5px;
        cursor: pointer;
        transition: .3s;
        width: 70px;
        text-align: center;
    }

    .btn-next {
        width: 95px;
    }

    .input-delay {
        cursor: text;
        width: 45px;
    }

    .btn:hover {
        background-color: rgb(225, 225, 225);
    }

    .btn-disabled {
        color: rgb(100, 100, 100);
        background-color: rgb(225, 225, 225);
        cursor: default;
    }

    .empty-space {
        background-color: rgba(0, 0, 0, 0.2);
        width: 95px;
        border-radius: 5px;
        border: 1px solid black;
    }

    .board {
        margin-top: 1em;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 5px;
    }

    .row {
        justify-content: center;
        align-items: center;
        display: flex;
        gap: 5px;
    }

    .box {
        width: 20px;
        height: 20px;
        background-color: rgb(100, 100, 100);
        transition: 0.15s;
        border-radius: 3px;
    }

    .box:hover {
        cursor: pointer;
        scale: 1.2;
    }

    .box-alive {
        background-color: white;
    }

</style>
