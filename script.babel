const style = getComputedStyle(document.documentElement);
const COLORS = {
  CURRENT: style.getPropertyValue("--color-current"),
  VISITED: style.getPropertyValue("--color-visited"),
  UNVISITED: style.getPropertyValue("--color-unvisited")
};

const COLS = 20;
const ROWS = 20;
const WIDTH = 400;
const HEIGHT = 400;
const CELL_SIZE = {
  x: parseInt(WIDTH / COLS),
  y: parseInt(HEIGHT / ROWS)
};

let stack = [];
const grid = new Array(COLS * ROWS);
const c = document.createElement("canvas");
const ctx = c.getContext("2d");
const scale = 2;

function Cell(x, y) {
  // walls: n,s,e,w
  return { x, y, walls: [1, 1, 1, 1] };
}

const oppositeWall = new Map([
  [0, 1], // north => south
  [1, 0], // south => north
  [2, 3], // east => west
  [3, 2] // west => east
]);

function clearGrid() {
  ctx.clearRect(0, 0, c.width, c.height);
}

// Draw the grid without taking cell walls into consideration
function drawGrid() {
  ctx.rect(0, 0, c.width, c.height);
  ctx.stroke();

  for (let x = 0; x < COLS; x++) {
    const ecks = x * (CELL_SIZE.x * scale) - 0.5;
    ctx.beginPath();
    ctx.moveTo(ecks, 0);
    ctx.lineTo(ecks, c.height);
    ctx.closePath();
    ctx.stroke();
  }

  for (let y = 0; y < ROWS; y++) {
    const why = y * (CELL_SIZE.y * scale) - 0.5;
    ctx.beginPath();
    ctx.moveTo(0, why);
    ctx.lineTo(c.width, why);
    ctx.closePath();
    ctx.stroke();
  }
}

function getNeighbors(x, y) {
  const north = y > 0 ? grid[x + (y - 1) * ROWS] : null;
  const south = y < ROWS - 1 ? grid[x + (y + 1) * ROWS] : null;
  const east = x < COLS - 1 ? grid[x + 1 + y * ROWS] : null;
  const west = x > 0 ? grid[x - 1 + y * ROWS] : null;
  return [north, south, east, west];
}

function clearWall(a, b) {
  // find the common wall index between a and b
  let wallA;
  if (a.y === b.y) {
    wallA = a.x === b.x - 1 ? 2 : 3;
  } else {
    wallA = a.y === b.y - 1 ? 1 : 0;
  }
  let wallB = oppositeWall.get(wallA);

  a.walls[wallA] = 0;
  b.walls[wallB] = 0;
}

function colorCell(cell, color) {
  ctx.fillStyle = color;
  const { x, y } = cell;
  ctx.fillRect(
    x * CELL_SIZE.x * scale,
    y * CELL_SIZE.y * scale,
    CELL_SIZE.x * scale,
    CELL_SIZE.y * scale
  );

  for (let i = 0; i < 4; i++) {
    const wall = cell.walls[i];
    if (!wall) continue;
    ctx.beginPath();
    switch (i) {
      case 0:
        ctx.moveTo(x * CELL_SIZE.x * scale, y * CELL_SIZE.y * scale);
        ctx.lineTo(
          x * CELL_SIZE.x * scale + CELL_SIZE.x * scale,
          y * CELL_SIZE.y * scale
        );
        break;
      case 1:
        ctx.moveTo(
          x * CELL_SIZE.x * scale,
          y * CELL_SIZE.y * scale + CELL_SIZE.y * scale
        );
        ctx.lineTo(
          x * CELL_SIZE.x * scale + CELL_SIZE.x * scale,
          y * CELL_SIZE.y * scale + CELL_SIZE.y * scale
        );
        break;
      case 2:
        ctx.moveTo(
          x * CELL_SIZE.x * scale + CELL_SIZE.x * scale,
          y * CELL_SIZE.y * scale
        );
        ctx.lineTo(
          x * CELL_SIZE.x * scale + CELL_SIZE.x * scale,
          y * CELL_SIZE.y * scale + CELL_SIZE.y * scale
        );
        break;
      case 3:
        ctx.moveTo(x * CELL_SIZE.x * scale, y * CELL_SIZE.y * scale);
        ctx.lineTo(
          x * CELL_SIZE.x * scale,
          y * CELL_SIZE.y * scale + CELL_SIZE.y * scale
        );
    }
    ctx.closePath();
    ctx.stroke();
  }
}

function chooseStartCell(x = 0, y = 0) {
  const startingCell = grid[x + y * ROWS];
  startingCell.visited = true;
  stack.push(startingCell);
}

function reset(_) {
  if (rafID) cancelAnimationFrame(rafID);

  stack = [];
  clearGrid();

  for (let x = 0; x < COLS; x++) {
    for (let y = 0; y < ROWS; y++) {
      grid[x + y * ROWS] = new Cell(x, y);
    }
  }

  chooseStartCell();
}

let rafID;

function animate() {
  step();
  if (stack.length > 0) rafID = requestAnimationFrame(animate);
  else cancelAnimationFrame(rafID);
}

function visualizer(_) {
  reset();
  rafID = requestAnimationFrame(animate);
}

function generate(_) {
  reset();
  while (stack.length) {
    step();
  }
}

// the algorithm
function step(_) {
  if (rafID) cancelAnimationFrame(rafID);

  const currentCell = stack.pop();
  const { x, y } = currentCell;
  const neighbors = getNeighbors(x, y).filter((n) => !!n);

  const unvisited = neighbors.filter((n) => !n.visited);

  clearGrid();

  // color visited cells, current cell
  grid.filter((c) => c.visited).forEach((c) => colorCell(c, COLORS.VISITED));
  colorCell(currentCell, COLORS.CURRENT);

  if (unvisited.length) {
    stack.push(currentCell);
    const nextCell = unvisited[Math.floor(Math.random() * unvisited.length)];
    nextCell.visited = true;
    stack.push(nextCell);

    clearWall(currentCell, nextCell);

    // color neightbor cells
    unvisited.forEach((c, i) => colorCell(c, COLORS.UNVISITED));
  }
}

// -----------------------------
// Initialization

c.width = WIDTH * scale;
c.height = HEIGHT * scale;
ctx.strokeStyle = "black";
ctx.lineWidth = scale;

document.querySelector("#stage").prepend(c);

reset();

// -----------------------------
// UI

const buttons = document.querySelectorAll("[data-action]");
const actions = { generate, reset, step, visualizer };

for (const button of buttons) {
  button.addEventListener("click", actions[button.dataset.action]);
}
