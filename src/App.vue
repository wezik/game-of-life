<script setup>

import { ref } from 'vue';

class Board {
  constructor(width, height) {
    this.width = width;
    this.height = height;
    this.cells = [];
    for (let y = 0; y < this.height; y++) {
      let line = [];
      for (let x = 0; x < this.width; x++) {
        let cell = new Cell(y * this.width + x, x, y, false);
        line.push(cell);
      }
      this.cells.push(line);
    }
  }
}

class Button {
  constructor(message) {
    this.disabled = false;
    this.message = message;
  }
}

class Cell {
  constructor(id, x, y, isAlive) {
    this.id = id;
    this.x = x;
    this.y = y;
    this.isAlive = isAlive;
  }

  copy() {
    return new Cell(this.id, this.x, this.y, this.isAlive);
  }
}

const widthInput = ref(new Button(20));
const heightInput = ref(new Button(20));
const state = ref(getInitialState());

const msDelayInput = ref(new Button(200));

const runButtonStartMessage = "START";
const runButtonStopMessage = "STOP";
const cycleButtonMessage = "CYCLE";
const restartButtonMessage = "RESTART";

const runButton = ref(new Button(runButtonStartMessage));
const cycleButton = ref(new Button(cycleButtonMessage));
const restartButton = ref(new Button(restartButtonMessage));

const activeCells = new Map();

function getInitialState() {
  return {
    isRunning: false,
    board: new Board(widthInput.value.message, heightInput.value.message) 
  }
}

function restartState() {
  if (state.value.isRunning) {
    changeState();
  }
  state.value.board = new Board(widthInput.value.message, heightInput.value.message);
}

let interval = null;

function changeState() {
  state.value.isRunning = !state.value.isRunning;
  let currentState = state.value.isRunning;
  interval = currentState ? setInterval(runCycle, msDelayInput.value.message) : clearInterval(interval);
  
  widthInput.value.disabled = currentState;
  heightInput.value.disabled = currentState;
  runButton.value.message = currentState ? runButtonStopMessage : runButtonStartMessage;
  msDelayInput.value.disabled = currentState;
  cycleButton.value.disabled = currentState;
}

function runCycle() {
  let board = state.value.board;
  let newBoard = new Board(board.width, board.height);
  let activeCellsCopy = new Map(activeCells);
  for (let cell of activeCells.values()) {
    addNeighboursToActivePull(cell.x, cell.y, activeCellsCopy);
  }
  activeCells.clear();
  for (let cell of activeCellsCopy.values()) {
    setAlive(newBoard.cells[cell.y][cell.x], calculateLife(cell.x, cell.y));
  }
  state.value.board = newBoard;
}

function calculateLife(x, y) {
  let board = state.value.board;
  let sum = countNeighbours(x, y);

  let stayAliveCondition = sum == 2 && board.cells[y][x].isAlive;
  let bornCondition = sum == 3;

  return stayAliveCondition || bornCondition;
}

function addNeighboursToActivePull(x, y, activeCellsPull) {
  for (let yOffset = -1; yOffset < 2; yOffset++) {
    for (let xOffset = -1; xOffset < 2; xOffset++) {
      if (xOffset != 0 || yOffset != 0) {
        let resultX = x + xOffset;
        let resultY = y + yOffset;
        if (!isOutOfBounds(resultX, resultY)) {
          let cell = state.value.board.cells[resultY][resultX];
          activeCellsPull.set(cell.id, cell.copy());
        }
      }
    }
  }
}

function countNeighbours(x, y) {
  let sum = 0;
  for (let yOffset = -1; yOffset < 2; yOffset++) {
    for (let xOffset = -1; xOffset < 2; xOffset++) {
      if (xOffset != 0 || yOffset != 0) {
        let resultX = x + xOffset;
        let resultY = y + yOffset;
        if (!isOutOfBounds(resultX, resultY)) {
          sum += checkCellIsAlive(resultX, resultY) ? 1 : 0;
        }
      }
    }
  }
  return sum;
}

function isOutOfBounds(x, y) {
  let board = state.value.board;

  let yIsOutOfBounds = y < 0 || y >= board.cells.length;
  if (yIsOutOfBounds) return true;

  let xIsOutOfBounds = x < 0 || x >= board.cells[y].length;
  if (xIsOutOfBounds) return true;

  return false;
}

function checkCellIsAlive(x, y) {
  return state.value.board.cells[y][x].isAlive;
}

function setAlive(cell, bool) {
  cell.isAlive = bool;
  cell.isAlive ? activeCells.set(cell.id, cell.copy()) : activeCells.delete(cell.id);
}

</script>

<template>
  <header>
    <input class="number-input" type="text" :disabled="widthInput.disabled" v-model="widthInput.message">
    <input class="number-input" type="text" :disabled="heightInput.disabled" v-model="heightInput.message">
    <button class="btn" @click="restartState" :disabled="restartButton.disabled">{{ restartButton.message }}</button>
    <button class="btn" @click="changeState" :disabled="runButton.disabled">{{ runButton.message }}</button>
    <input class="number-input" type="text" :disabled="msDelayInput.disabled" v-model="msDelayInput.message">
    <button class="btn" @click="runCycle" :disabled="cycleButton.disabled">{{ cycleButton.message }}</button>
  </header>
  <main>
    <div class="board">
      <div class="row" v-for="row in state.board.cells">
        <div class="cell" v-for="cell in row" @click="setAlive(cell,!cell.isAlive)" :class="{ 'cell--alive': cell.isAlive }">
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
