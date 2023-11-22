<script setup>

import { ref } from 'vue';

class Board {
  constructor(width, height) {
    this.width = width;
    this.height = height;
    this.cells = this.createEmptyCells();
  }

  createEmptyCells() {
    const cells = [];
    for (let y = 0; y < this.height; y++) {
      const row = [];
      for (let x = 0; x < this.width; x++) {
        const cell = new Cell(y * this.width + x, x, y, false);
        row.push(cell);
      }
      cells.push(row);
    }
    return cells;
  }
}

class Button {
  constructor(label) {
    this.disabled = false;
    this.label = label;
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
const state = ref(initialState());

const msDelayInput = ref(new Button(200));

const runButtonStartLabel = "START";
const runButtonStopLabel = "STOP";
const cycleButtonLabel = "CYCLE";
const restartButtonLabel = "RESTART";

const runButton = ref(new Button(runButtonStartLabel));
const cycleButton = ref(new Button(cycleButtonLabel));
const restartButton = ref(new Button(restartButtonLabel));

const activeCells = new Map();

function initialState() {
  return {
    isRunning: false,
    board: new Board(widthInput.value.label, heightInput.value.label)
  };
}

function restartState() {
  if (state.value.isRunning) {
    changeState();
  }
  state.value.board = new Board(widthInput.value.label, heightInput.value.label);
  activeCells.clear();
}

let interval = null;

function changeState() {
  state.value.isRunning = !state.value.isRunning;
  const currentState = state.value.isRunning;
  interval = currentState ? setInterval(runCycle, msDelayInput.value.label) : clearInterval(interval);
  
  widthInput.value.disabled = currentState;
  heightInput.value.disabled = currentState;
  runButton.value.label = currentState ? runButtonStopLabel : runButtonStartLabel;
  msDelayInput.value.disabled = currentState;
  cycleButton.value.disabled = currentState;
}

let bufferBoard = null;

function runCycle() {
  restartBufferBoard();

  const activeCellsCopy = new Map(activeCells);

  for (const cell of activeCells.values()) {
    addNeighboursToActivePull(cell.x, cell.y, activeCellsCopy);
  }

  activeCells.clear();

  for (const cell of activeCellsCopy.values()) {
    setAlive(bufferBoard.cells[cell.y][cell.x], calculateLife(cell.x, cell.y));
  }

  for (const cell of activeCellsCopy.values()) {
    console.log(`Active cell: X${cell.x} Y${cell.y}`)
    state.value.board.cells[cell.y][cell.x].isAlive = bufferBoard.cells[cell.y][cell.x].isAlive;
  }
}

function restartBufferBoard() {
  if (!bufferBoard) {
    bufferBoard = new Board(state.value.board.width, state.value.board.height);
  }
  for (const cell of activeCells.values()) {
    bufferBoard.cells[cell.y][cell.x].isAlive = false;
  }
}

function calculateLife(x, y) {
  const board = state.value.board;
  const sum = countNeighbours(x, y);

  const stayAliveCondition = sum === 2 && board.cells[y][x].isAlive;
  const bornCondition = sum === 3;

  return stayAliveCondition || bornCondition;
}

function addNeighboursToActivePull(x, y, activeCellsPull) {
  for (let yOffset = -1; yOffset < 2; yOffset++) {
    for (let xOffset = -1; xOffset < 2; xOffset++) {
      if (xOffset !== 0 || yOffset !== 0) {
        const resultX = x + xOffset;
        const resultY = y + yOffset;
        if (!isOutOfBounds(resultX, resultY)) {
          const cell = state.value.board.cells[resultY][resultX];
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
      if (xOffset !== 0 || yOffset !== 0) {
        const resultX = x + xOffset;
        const resultY = y + yOffset;
        if (!isOutOfBounds(resultX, resultY)) {
          sum += checkCellIsAlive(resultX, resultY) ? 1 : 0;
        }
      }
    }
  }
  return sum;
}

function isOutOfBounds(x, y) {
  const board = state.value.board;

  const yIsOutOfBounds = y < 0 || y >= board.cells.length;
  if (yIsOutOfBounds) return true;

  const xIsOutOfBounds = x < 0 || x >= board.cells[y].length;
  if (xIsOutOfBounds) return true;

  return false;
}

function checkCellIsAlive(x, y) {
  return state.value.board.cells[y][x].isAlive;
}

function setAlive(cell, bool) {
  cell.isAlive = bool;
  bool ? activeCells.set(cell.id, cell.copy()) : activeCells.delete(cell.id);
}

</script>

<template>
  <header>
    <input class="number-input" type="text" :disabled="widthInput.disabled" v-model="widthInput.label">
    <input class="number-input" type="text" :disabled="heightInput.disabled" v-model="heightInput.label">
    <button class="btn" @click="restartState" :disabled="restartButton.disabled">{{ restartButton.label }}</button>
    <button class="btn" @click="changeState" :disabled="runButton.disabled">{{ runButton.label }}</button>
    <input class="number-input" type="text" :disabled="msDelayInput.disabled" v-model="msDelayInput.label">
    <button class="btn" @click="runCycle" :disabled="cycleButton.disabled">{{ cycleButton.label }}</button>
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
