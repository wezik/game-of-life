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

const activeCellsId = new Set();

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
  activeCellsId.clear();
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

function runCycle() {
  // Calculate next gen active cells
  const activeCellsNextGen = calculateLife(activeCellsId);

  // Clear active cells id set
  activeCellsId.clear();

  // Write active cells to board and map new activeCellsIdSet
  for (const cell of activeCellsNextGen) {
    const displayedCell = state.value.board.cells[cell.y][cell.x];
    setAlive(displayedCell, cell.isAlive);
  }

}

function calculateLife(activeCellsId) {
  //Create buffer, map id to cells and pull neighbours
  const activeCellsNextGen = createNextGenSet(activeCellsId);

  //Calculate all activeCells
  calculateCells(activeCellsNextGen);
  return activeCellsNextGen;
}

function calculateCells(activeCellsNextGen) {
  for (const cell of activeCellsNextGen) {
    const sum = getSumOfNeighbours(cell);
    const stayAliveCondition = sum == 2 && cell.isAlive;
    const bornCondition = sum == 3; 
    cell.isAlive = stayAliveCondition || bornCondition;
  }
}

function getSumOfNeighbours(cell) {
  let sum = 0;
  for (let yOffset = -1; yOffset < 2; yOffset++) {
    for (let xOffset = -1; xOffset < 2; xOffset++) {
      const resultX = cell.x + xOffset;
      const resultY = cell.y + yOffset;
      if (!isOutOfBounds(resultX, resultY)) {
        if (resultX != cell.x || resultY != cell.y) {
          sum += state.value.board.cells[resultY][resultX].isAlive ? 1 : 0;
        }
      }
    }
  }
  return sum;
}

function createNextGenSet(activeCells) {
  const resultSet = new Set();
  for (const cellId of activeCells.values()) {
    const cell = getCellById(cellId);
    pullNeighbours(cell, resultSet);
  }
  return resultSet;
}

function pullNeighbours(cell, set) {
  for (let yOffset = -1; yOffset < 2; yOffset++) {
    for (let xOffset = -1; xOffset < 2; xOffset++) {
      const resultX = cell.x + xOffset;
      const resultY = cell.y + yOffset;
      if (!isOutOfBounds(resultX, resultY)) {
        const cell = state.value.board.cells[resultY][resultX].copy();
        addCellToSet(cell, set);
      }
    }
  }
}

function addCellToSet(cell, set) {
  for (const obj of set) {
    if (cell.id === obj.id) {
      return;
    }
  }
  set.add(cell);
}

function getCellById(id) {
    const calculatedX = id % state.value.board.width;
    const calculatedY = (id - calculatedX) / state.value.board.height;
    return state.value.board.cells[calculatedY][calculatedX].copy();
  }

function isOutOfBounds(x, y) {
  const board = state.value.board;

  const yIsOutOfBounds = y < 0 || y >= board.cells.length;
  if (yIsOutOfBounds) return true;

  const xIsOutOfBounds = x < 0 || x >= board.cells[y].length;
  if (xIsOutOfBounds) return true;

  return false;
}

function setAlive(cell, bool) {
  cell.isAlive = bool;
  bool ? activeCellsId.add(cell.id) : activeCellsId.delete(cell.id);
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
    color: black;
  }

  .cell--alive {
    background-color: white;
  }

  .cell:hover {
    cursor: pointer;
  }

</style>
