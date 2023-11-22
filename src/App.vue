<script setup>

import { ref } from 'vue';

class Button {
  constructor(message) {
    this.disabled = false;
    this.message = message;
  }
}

class Cell {
  constructor(id, isAlive) {
    this.id = id;
    this.isAlive = isAlive;
  }
}

const widthInput = ref(20);
const heightInput = ref(20);
const state = ref(getInitialState());

const msDelayInput = ref(200);

const runButtonStartMessage = "START";
const runButtonStopMessage = "STOP";
const cycleButtonMessage = "CYCLE";
const restartButtonMessage = "RESTART";

const runButton = ref(new Button(runButtonStartMessage));
const cycleButton = ref(new Button(cycleButtonMessage));
const restartButton = ref(new Button(restartButtonMessage));

function getInitialState() {
  return {
    isRunning: false,
    board: createEmptyBoard() 
  }
}

function createEmptyBoard() {
  let board = [];
  for (let y = 0; y < heightInput.value; y++) {
    let row = [];
    for (let x = 0; x < widthInput.value; x++) {
      let cell = new Cell(y * widthInput.value + x, false);
      row.push(cell);
    }
    board.push(row);
  }
  return board;
}

function restartState() {
  if (state.value.isRunning) {
    changeState();
  }
  state.value.board = createEmptyBoard();
}

let interval = null;

function changeState() {
  state.value.isRunning = !state.value.isRunning;
  let currentState = state.value.isRunning;
  interval = currentState ? setInterval(runCycle, msDelayInput.value) : clearInterval(interval);
  
  runButton.value.message = currentState ? runButtonStopMessage : runButtonStartMessage;
  cycleButton.value.disabled = currentState;
}

function runCycle() {
  let board = state.value.board;
  let newBoard = [];
  for (let y = 0; y < board.length; y++) {
    let newRow = [];
    for (let x = 0; x < board[y].length; x++) {
      let cell = new Cell(board[y][x].id, calculateLife(x, y));
      newRow[x] = cell;
    }
    newBoard.push(newRow);
  }
  state.value.board = newBoard;
}

function calculateLife(x, y) {
  let board = state.value.board;
  let sum = countNeighbours(x, y);

  let stayAliveCondition = sum == 2 && board[y][x].isAlive;
  let bornCondition = sum == 3;

  return stayAliveCondition || bornCondition;
}

function countNeighbours(x, y) {
  let sum = 0;
  for (let yOffset = -1; yOffset < 2; yOffset++) {
    for (let xOffset = -1; xOffset < 2; xOffset++) {
      if (xOffset != 0 || yOffset != 0) {
        sum += checkCellIsAlive(x + xOffset, y + yOffset) ? 1 : 0;
      }
    }
  }
  return sum;
}

function checkCellIsAlive(x, y) {
  let board = state.value.board;

  let yIsOutOfBounds = y < 0 || y >= board.length;
  if (yIsOutOfBounds) return false;

  let xIsOutOfBounds = x < 0 || x >= board[y].length;
  if (xIsOutOfBounds) return false;

  return board[y][x].isAlive;
}

</script>

<template>
  <header>
    <input class="number-input" type="text" v-model="widthInput">
    <input class="number-input" type="text" v-model="heightInput">
    <button class="btn" @click="restartState" :disabled="restartButton.disabled">{{ restartButton.message }}</button>
    <button class="btn" @click="changeState" :disabled="runButton.disabled">{{ runButton.message }}</button>
    <input class="number-input" type="text" v-model="msDelayInput">
    <button class="btn" @click="runCycle" :disabled="cycleButton.disabled">{{ cycleButton.message }}</button>
  </header>
  <main>
    <div class="board">
      <div class="row" v-for="row in state.board">
        <div class="cell" v-for="cell in row" @click="cell.isAlive = !cell.isAlive" :class="{ 'cell--alive': cell.isAlive }">
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
  header {
    padding-top: 2em;
    display: flex;
    justify-content: center;
    gap: 5px;
  }

  main {
    margin-top: 2em;
    display: flex;
    justify-content: center;
  }

  .btn {
    height: 2em;
    font-weight: 600;
  }

  .number-input {
    height: 2em;
    font-weight: 600;
    width: 5em;
  }
  
  .board {
    display: flex;
    flex-direction: column;
  }

  .row {
    display: flex;
    flex-direction: row;
  }

  .cell {
    height: 20px;
    width: 20px;
    background-color: rgb(50, 50, 50);
    transition: .15s;
    border-radius: 5px;
    border: 1px solid black;
  }

  .cell--alive {
    background-color: white;
  }

  .cell:hover {
    cursor: pointer;
  }

</style>
