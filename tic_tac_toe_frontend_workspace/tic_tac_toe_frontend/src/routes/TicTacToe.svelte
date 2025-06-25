<script lang="ts">
// PUBLIC_INTERFACE
/**
 * Main Tic Tac Toe game component.
 * Features:
 *  - Player vs Player and Player vs Computer
 *  - Responsive, modern/minimal layout
 *  - Uses provided color palette: primary/accent/secondary
 *  - Displays board, status, start/new game and restart controls
 */

import { onMount } from 'svelte';

type Player = 'X' | 'O';
type Cell = Player | '';
type Board = Cell[][];
type GameMode = 'pvp' | 'cpu';

const BOARD_SIZE = 3;

// --- State ---
let board: Board = [];
let currentPlayer: Player = 'X';
let winner: Player | null = null;
let draw: boolean = false;
let started: boolean = false;
let gameMode: GameMode = 'pvp';
let status: string = '';
let gameActive: boolean = true;
let startingPlayer: Player = 'X';

// --- Setup Color Palette ---
const PALETTE = {
  accent: '#fbbf24',    // yellow
  primary: '#2563eb',   // blue
  secondary: '#64748b', // slate
};

// --- New Game Initialization ---
function initBoard() {
  board = Array.from({ length: BOARD_SIZE }, () => Array<Cell>(BOARD_SIZE).fill(''));
  winner = null;
  draw = false;
  currentPlayer = startingPlayer;
  gameActive = true;
  status = '';
  started = true;
  // If CPU starts, let it make first move
  if (gameMode === 'cpu' && currentPlayer === 'O') {
    setTimeout(() => computerMove(), 400);
  }
}

function startNewGame(selectedMode: GameMode) {
  gameMode = selectedMode;
  startingPlayer = 'X';
  initBoard();
}

function restartGame() {
  // Alternate the starting player for fairness
  startingPlayer = startingPlayer === 'X' ? 'O' : 'X';
  initBoard();
}

// --- Board Actions & Logic ---
function handleCellClick(row: number, col: number) {
  if (!gameActive || board[row][col] !== '') return;
  if (winner || draw) return;
  if (gameMode === 'cpu' && currentPlayer === 'O') return; // wait for CPU

  board[row][col] = currentPlayer;
  checkGameState();

  if (gameMode === 'cpu' && currentPlayer === 'O' && !winner && !draw) {
    setTimeout(() => computerMove(), 400); // CPU 'O' move
  }
}

function checkGameState() {
  const result = calculateWinner(board);
  if (result) {
    winner = result;
    gameActive = false;
    status = `${winner} wins! 🎉`;
    return;
  }
  if (isBoardFull(board)) {
    draw = true;
    gameActive = false;
    status = "It's a draw! 🤝";
    return;
  }
  currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
  status = `${currentPlayer}'s turn` + (gameMode === 'cpu' && currentPlayer === 'O' ? ' (Computer)' : '');
}

// Returns 'X' | 'O' | null
function calculateWinner(board: Board): Player | null {
  // Rows, columns, diagonals
  for (let i = 0; i < BOARD_SIZE; i++) {
    // Row
    if (board[i][0] && board[i][0] === board[i][1] && board[i][1] === board[i][2]) return board[i][0];
    // Column
    if (board[0][i] && board[0][i] === board[1][i] && board[1][i] === board[2][i]) return board[0][i];
  }
  // Diagonals
  if (board[0][0] && board[0][0] === board[1][1] && board[1][1] === board[2][2]) return board[0][0];
  if (board[0][2] && board[0][2] === board[1][1] && board[1][1] === board[2][0]) return board[0][2];
  return null;
}

function isBoardFull(board: Board): boolean {
  return board.every(row => row.every(cell => cell !== ''));
}

// ----- Computer AI (Random available spot, could be improved) -----
function computerMove() {
  if (!gameActive || currentPlayer !== 'O') return;
  // Collect empty cells
  const empties: { row: number, col: number }[] = [];
  board.forEach((row, i) => row.forEach((cell, j) => {
    if (cell === '') empties.push({ row: i, col: j });
  }));
  if (empties.length === 0) return;

  // Try to win if possible, otherwise block X, else random
  const { row, col } = findBestMove('O') || findBestMove('X') || empties[Math.floor(Math.random() * empties.length)];
  board[row][col] = 'O';
  checkGameState();
}

// AI Helper: Find winning/blocking move for player
function findBestMove(player: Player): { row: number, col: number } | null {
  for (let i = 0; i < BOARD_SIZE; i++) {
    for (let j = 0; j < BOARD_SIZE; j++) {
      if (board[i][j] === '') {
        board[i][j] = player;
        if (calculateWinner(board) === player) {
          board[i][j] = ''; // revert
          return { row: i, col: j };
        }
        board[i][j] = '';
      }
    }
  }
  return null;
}

onMount(() => {
  board = [];
  started = false;
});
</script>

<!-- UI -->
<div class="ttt-root">
  <div class="ttt-container">
    <h1 class="ttt-title">Tic Tac Toe</h1>
    {#if !started}
      <div class="mode-select">
        <p>Select Game Mode:</p>
        <div class="mode-btns">
          <button
            class="ttt-btn mode"
            on:click={() => startNewGame('pvp')}
            style="background: {PALETTE.primary};"
            aria-label="Play against another player"
          >Player vs Player</button>
          <button
            class="ttt-btn mode"
            on:click={() => startNewGame('cpu')}
            style="background: {PALETTE.secondary};"
            aria-label="Play against computer"
          >Player vs Computer</button>
        </div>
      </div>
    {:else}
      <div class="ttt-controls">
        <div>
          <span class="label">Mode:</span>
          <span>{gameMode === 'pvp' ? "Player vs Player" : "Player vs Computer"}</span>
        </div>
        <div>
          <button class="ttt-btn small" on:click={restartGame} aria-label="Restart Game">Restart</button>
        </div>
      </div>
      <div class="ttt-status"
        style="color: {winner ? PALETTE.accent : (draw ? PALETTE.secondary : PALETTE.primary)};">
        {#if winner}
          <span class="status-text">{status}</span>
        {:else if draw}
          <span class="status-text">{status}</span>
        {:else}
          <span class="status-text">{status}</span>
        {/if}
      </div>

      <div class="ttt-board" role="grid" aria-label="Tic Tac Toe board">
        {#each board as row, i}
          <div class="ttt-row" role="row">
            {#each row as cell, j}
              <button
                class="ttt-cell"
                role="gridcell"
                aria-label="Cell {i + 1}, {j + 1} {cell ? (cell==='X' ? 'X' : 'O') : ''}"
                on:click={() => handleCellClick(i, j)}
                disabled={!!cell || winner || draw || (gameMode === 'cpu' && currentPlayer === 'O')}
                style="color: {cell === 'X' ? PALETTE.primary : cell === 'O' ? PALETTE.accent : PALETTE.secondary};"
              >
                {cell}
              </button>
            {/each}
          </div>
        {/each}
      </div>
    {/if}
    <div class="ttt-footer">
      <button
        class="ttt-btn outline"
        on:click={() => { started = false; board = []; }}
        aria-label="Start New Game"
        >
        New Game
      </button>
    </div>
  </div>
</div>

<style>
.ttt-root {
  min-height: 80vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: none;
}
.ttt-container {
  width: 100%;
  max-width: 350px;
  margin: 0 auto;
  background: #fff;
  border-radius: 22px;
  box-shadow: 0 8px 24px rgba(80,100,130,.07), 0 1.5px 4px rgba(50,60,70,.2);
  padding: 2.5rem 1.25rem 1.5rem 1.25rem;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  gap: 1.1rem;
}

.ttt-title {
  text-align: center;
  color: {PALETTE.primary};
  font-size: 2.1rem;
  font-weight: 700;
  margin: 0 0 4px 0;
  letter-spacing: 0.01em;
}

.mode-select {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.8rem;
  padding: 0.5rem 0;
}

.mode-btns {
  display: flex;
  gap: 1.1rem;
}

.ttt-btn {
  border: none;
  font-size: 1rem;
  font-weight: 600;
  background: {PALETTE.primary};
  color: #fff;
  padding: 0.65em 1.25em;
  border-radius: 8px;
  outline: none;
  cursor: pointer;
  transition: box-shadow 0.15s, background 0.15s;
  margin: 3px 0;
  box-shadow: 0 2px 8px 0 rgba(20, 40, 70, 0.08);
}
.ttt-btn.mode {
  background: {PALETTE.secondary};
}
.ttt-btn.outline {
  background: none;
  color: {PALETTE.primary};
  border: 2px solid {PALETTE.primary};
  margin: 0 auto;
  display: block;
}
.ttt-btn.small {
  font-size: 0.9rem;
  padding: 0.45em 0.9em;
  background: {PALETTE.secondary};
}

.ttt-btn:hover, .ttt-btn:focus {
  box-shadow: 0 4px 12px 0 rgba(55, 120, 255, 0.13);
  background: {PALETTE.primary};
  color: #fff;
}

.ttt-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 1rem;
  margin-bottom: 0.4em;
}

.label {
  font-weight: 600;
  color: {PALETTE.secondary};
  margin-right: 0.2em;
}

.ttt-status {
  min-height: 2.1em;
  text-align: center;
  font-size: 1.14rem;
  margin: 0.6em 0 0.5em 0;
}

.status-text {
  font-weight: 700;
  letter-spacing: 0.01em;
}

.ttt-board {
  display: grid;
  grid-template-rows: repeat(3, 1fr);
  gap: 0.46rem;
  width: 100%;
  aspect-ratio: 1 / 1;
  margin: 0 auto;
  max-width: 320px;
  background: #fafbfc;
  border-radius: 14px;
  box-shadow: 0 2px 12px 0 rgba(90,120,220,.05);
  padding: 0.5rem;
}

.ttt-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 0.44rem;
}

.ttt-cell {
  width: 100%;
  aspect-ratio: 1 / 1;
  font-size: 2.2rem;
  font-family: inherit;
  font-weight: 700;
  color: {PALETTE.primary};
  background: #fff;
  border: 2.5px solid {PALETTE.secondary};
  border-radius: 10px;
  outline: none;
  box-shadow: 0 1px 2px 0 rgba(50,60,70,.05);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: border-color 0.15s, background 0.15s;
  cursor: pointer;
  user-select: none;
  position: relative;
}
.ttt-cell:hover, .ttt-cell:focus {
  border-color: {PALETTE.accent};
  background: #fef7e6;
}
.ttt-cell:disabled {
  cursor: not-allowed;
  opacity: 0.7;
  background: #f7fafc;
}

.ttt-footer {
  margin-top: 1.2em;
  text-align: center;
}

@media (max-width: 520px) {
  .ttt-container {
    padding: 1.4rem 0.4rem 1rem 0.4rem;
    max-width: 98vw;
  }
  .ttt-title {
    font-size: 1.5rem;
  }
  .ttt-board {
    padding: 0.35rem;
  }
  .ttt-cell {
    font-size: 1.35rem;
  }
}
</style>
