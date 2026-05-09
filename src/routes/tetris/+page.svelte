<svelte:options runes={true} />

<script lang="ts">
	import BackHome from '$lib/components/BackHome.svelte';
	import { onMount } from 'svelte';

	const COLS = 10;
	const ROWS = 20;
	const BLOCK_SIZE = 30;
	const WHITE_COLOR_ID = 7;

	const COLOR_MAPPING: string[] = [
		'#ff4655',
		'#ff9f1c',
		'#2ec4b6',
		'#c77dff',
		'#4895ef',
		'#00f5d4',
		'#ffe548',
		'#1a1a2e',
	];

	const GLOW_MAPPING: string[] = [
		'rgba(255,70,85,0.6)',
		'rgba(255,159,28,0.6)',
		'rgba(46,196,182,0.6)',
		'rgba(199,125,255,0.6)',
		'rgba(72,149,239,0.6)',
		'rgba(0,245,212,0.6)',
		'rgba(255,229,72,0.6)',
		'transparent',
	];

	const BRICK_LAYOUT: number[][][][] = [
		[ 
			[[1,7,7],[1,1,1],[7,7,7]],
			[[7,1,1],[7,1,7],[7,1,7]],
			[[7,7,7],[1,1,1],[7,7,1]],
			[[7,1,7],[7,1,7],[1,1,7]],
		],
		[ 
			[[7,1,7],[7,1,7],[7,1,1]],
			[[7,7,7],[1,1,1],[1,7,7]],
			[[1,1,7],[7,1,7],[7,1,7]],
			[[7,7,1],[1,1,1],[7,7,7]],
		],
		[
			[[1,7,7],[1,1,7],[7,1,7]],
			[[7,1,1],[1,1,7],[7,7,7]],
			[[7,1,7],[7,1,1],[7,7,1]],
			[[7,7,7],[7,1,1],[1,1,7]],
		],
		[
			[[7,1,7],[1,1,7],[1,7,7]],
			[[1,1,7],[7,1,1],[7,7,7]],
			[[7,7,1],[7,1,1],[7,1,7]],
			[[7,7,7],[1,1,7],[7,1,1]],
		],
		[
			[[7,7,7,7],[1,1,1,1],[7,7,7,7],[7,7,7,7]],
			[[7,7,1,7],[7,7,1,7],[7,7,1,7],[7,7,1,7]],
			[[7,7,7,7],[7,7,7,7],[1,1,1,1],[7,7,7,7]],
			[[7,1,7,7],[7,1,7,7],[7,1,7,7],[7,1,7,7]],
		],
		[
			[[7,7,7,7],[7,1,1,7],[7,1,1,7],[7,7,7,7]],
			[[7,7,7,7],[7,1,1,7],[7,1,1,7],[7,7,7,7]],
			[[7,7,7,7],[7,1,1,7],[7,1,1,7],[7,7,7,7]],
			[[7,7,7,7],[7,1,1,7],[7,1,1,7],[7,7,7,7]],
		],
		[
			[[7,1,7],[1,1,1],[7,7,7]],
			[[7,1,7],[7,1,1],[7,1,7]],
			[[7,7,7],[1,1,1],[7,1,7]],
			[[7,1,7],[1,1,7],[7,1,7]],
		],
	];

	type Grid = number[][];

	let canvas = $state<HTMLCanvasElement | null>(null);
	let ctx    = $state<CanvasRenderingContext2D | null>(null);

	let grid      = $state<Grid>(generateEmptyGrid());
	let score     = $state(0);
	let gameOver  = $state(false);
	let isPlaying = $state(false);
	let isPaused  = $state(false);
	let soundOn   = $state(true);

	let brickId      = $state(0);
	let activeIndex  = $state(0);
	let colPos       = $state(3);
	let rowPos       = $state(-2);

	let intervalId: ReturnType<typeof setInterval> | null = null;

	function generateEmptyGrid(): Grid {
		return Array.from({ length: ROWS }, () => Array(COLS).fill(WHITE_COLOR_ID));
	}

	function drawCell(x: number, y: number, colorId: number, ghost = false) {
		if (!ctx) return;
		const color = COLOR_MAPPING[colorId] ?? COLOR_MAPPING[WHITE_COLOR_ID];
		const glow  = GLOW_MAPPING[colorId]  ?? 'transparent';
		const px = x * BLOCK_SIZE;
		const py = y * BLOCK_SIZE;
		const bs = BLOCK_SIZE;

		if (colorId === WHITE_COLOR_ID) {
			ctx.fillStyle = '#0d0d1a';
			ctx.fillRect(px, py, bs, bs);
			ctx.strokeStyle = 'rgba(255,255,255,0.04)';
			ctx.lineWidth = 0.5;
			ctx.strokeRect(px + 0.25, py + 0.25, bs - 0.5, bs - 0.5);
			return;
		}

		if (ghost) {
			ctx.fillStyle = color + '30';
			ctx.fillRect(px + 1, py + 1, bs - 2, bs - 2);
			ctx.strokeStyle = color + '60';
			ctx.lineWidth = 1;
			ctx.strokeRect(px + 1, py + 1, bs - 2, bs - 2);
			return;
		}

		const grad = ctx.createLinearGradient(px, py, px + bs, py + bs);
		grad.addColorStop(0, lighten(color, 40));
		grad.addColorStop(1, color);
		ctx.fillStyle = grad;
		ctx.fillRect(px + 1, py + 1, bs - 2, bs - 2);

		ctx.shadowColor   = glow;
		ctx.shadowBlur    = 10;
		ctx.fillRect(px + 1, py + 1, bs - 2, bs - 2);
		ctx.shadowBlur = 0;

		ctx.fillStyle = 'rgba(255,255,255,0.15)';
		ctx.fillRect(px + 2, py + 2, bs - 4, 4);
	}

	function lighten(hex: string, amount: number): string {
		const num = parseInt(hex.replace('#', ''), 16);
		const r = Math.min(255, (num >> 16) + amount);
		const g = Math.min(255, ((num >> 8) & 0xff) + amount);
		const b = Math.min(255, (num & 0xff) + amount);
		return `rgb(${r},${g},${b})`;
	}

	function drawBoard() {
		if (!ctx || !canvas) return;
		ctx.clearRect(0, 0, canvas.width, canvas.height);
		for (let row = 0; row < ROWS; row++) {
			for (let col = 0; col < COLS; col++) {
				drawCell(col, row, grid[row][col]);
			}
		}
	}

	function getLayout() {
		return BRICK_LAYOUT[brickId][activeIndex];
	}

	function drawBrick(
		rid: number, aidx: number, cpos: number, rpos: number,
		colorId: number, ghost = false
	) {
		const layout = BRICK_LAYOUT[rid][aidx];
		for (let r = 0; r < layout.length; r++) {
			for (let c = 0; c < layout[0].length; c++) {
				if (layout[r][c] !== WHITE_COLOR_ID) {
					const gx = c + cpos;
					const gy = r + rpos;
					if (gy >= 0) drawCell(gx, gy, colorId, ghost);
				}
			}
		}
	}

	function getGhostRow(): number {
		let ghost = rowPos;
		while (!checkCollision(ghost + 1, colPos, getLayout())) ghost++;
		return ghost;
	}

	function renderFrame() {
		drawBoard();
		const ghostRow = getGhostRow();
		if (ghostRow !== rowPos) drawBrick(brickId, activeIndex, colPos, ghostRow, brickId, true);
		drawBrick(brickId, activeIndex, colPos, rowPos, brickId, false);
	}

	function checkCollision(nextRow: number, nextCol: number, layout: number[][]): boolean {
		for (let r = 0; r < layout.length; r++) {
			for (let c = 0; c < layout[0].length; c++) {
				if (layout[r][c] !== WHITE_COLOR_ID && nextRow >= 0) {
					if (
						c + nextCol < 0 ||
						c + nextCol >= COLS ||
						r + nextRow >= ROWS ||
						grid[r + nextRow][c + nextCol] !== WHITE_COLOR_ID
					) return true;
				}
			}
		}
		return false;
	}

	function moveLeft() {
		if (!isPlaying || gameOver || isPaused) return;
		if (!checkCollision(rowPos, colPos - 1, getLayout())) {
			colPos--;
			renderFrame();
		}
	}

	function moveRight() {
		if (!isPlaying || gameOver || isPaused) return;
		if (!checkCollision(rowPos, colPos + 1, getLayout())) {
			colPos++;
			renderFrame();
		}
	}

	function rotate() {
		if (!isPlaying || gameOver || isPaused) return;
		const nextIdx = (activeIndex + 1) % 4;
		if (!checkCollision(rowPos, colPos, BRICK_LAYOUT[brickId][nextIdx])) {
			activeIndex = nextIdx;
			renderFrame();
		}
	}

	function moveDown() {
		if (!isPlaying || gameOver || isPaused) return;
		if (!checkCollision(rowPos + 1, colPos, getLayout())) {
			rowPos++;
			renderFrame();
		} else {
			handleLanded();
		}
	}

	function handleLanded() {
		if (rowPos <= 0) {
			handleGameOver();
			return;
		}
		const layout = getLayout();
		const newGrid = grid.map(row => [...row]);
		for (let r = 0; r < layout.length; r++) {
			for (let c = 0; c < layout[0].length; c++) {
				if (layout[r][c] !== WHITE_COLOR_ID) {
					newGrid[r + rowPos][c + colPos] = brickId;
				}
			}
		}
		grid = newGrid;
		handleCompleteRows();
		generateNewBrick();
		renderFrame();
	}

	function handleCompleteRows() {
		const remaining = grid.filter(row => row.some(cell => cell === WHITE_COLOR_ID));
		const cleared   = ROWS - remaining.length;
		if (cleared > 0) {
			const newRows = Array.from({ length: cleared }, () => Array(COLS).fill(WHITE_COLOR_ID));
			grid  = [...newRows, ...remaining];
			score += cleared * 10;
			if (soundOn) playBeep();
		}
	}

	function generateNewBrick() {
		brickId     = Math.floor(Math.random() * BRICK_LAYOUT.length);
		activeIndex = 0;
		colPos      = 3;
		rowPos      = -2;
	}

	function handleGameOver() {
		gameOver  = true;
		isPlaying = false;
		if (intervalId) clearInterval(intervalId);
	}

	function playBeep() {
		try {
			const audioCtx = new AudioContext();
			const osc = audioCtx.createOscillator();
			const gain = audioCtx.createGain();
			osc.connect(gain);
			gain.connect(audioCtx.destination);
			osc.type = 'sine';
			osc.frequency.setValueAtTime(880, audioCtx.currentTime);
			osc.frequency.exponentialRampToValueAtTime(440, audioCtx.currentTime + 0.15);
			gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
			gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.2);
			osc.start();
			osc.stop(audioCtx.currentTime + 0.2);
		} catch (_) {}
	}

	function togglePause() {
		if (!isPlaying || gameOver) return;
		isPaused = !isPaused;
	}

	function startGame() {
		if (intervalId) clearInterval(intervalId);
		grid      = generateEmptyGrid();
		score     = 0;
		gameOver  = false;
		isPlaying = true;
		isPaused  = false;
		generateNewBrick();
		renderFrame();
		intervalId = setInterval(() => {
			if (!gameOver) moveDown();
			else if (intervalId) clearInterval(intervalId);
		}, 500);
	}

	function handleKeyDown(e: KeyboardEvent) {
		if (e.code === 'Space' && isPlaying && !gameOver) {
			e.preventDefault();
			togglePause();
			return;
		}
		if (!isPlaying || gameOver || isPaused) return;
		switch (e.code) {
			case 'ArrowLeft':  e.preventDefault(); moveLeft();  break;
			case 'ArrowRight': e.preventDefault(); moveRight(); break;
			case 'ArrowDown':  e.preventDefault(); moveDown();  break;
			case 'ArrowUp':    e.preventDefault(); rotate();    break;
		}
	}

	onMount(() => {
		if (!canvas) return;
		ctx = canvas.getContext('2d');
		canvas.width  = COLS * BLOCK_SIZE;
		canvas.height = ROWS * BLOCK_SIZE;
		drawBoard();

		window.addEventListener('keydown', handleKeyDown);
		return () => {
			window.removeEventListener('keydown', handleKeyDown);
			if (intervalId) clearInterval(intervalId);
		};
	});
</script>

<div class="app">
	<BackHome/>
	<header>
		<span class="logo">TETRIS</span>
		<button
			class="sound-btn"
			class:muted={!soundOn}
			onclick={() => soundOn = !soundOn}
			title={soundOn ? 'Tắt âm' : 'Bật âm'}
		>
			{soundOn ? '🔊' : '🔇'}
		</button>
	</header>

	<main>
		<!-- Board -->
		<div class="board-wrap">
			<canvas bind:this={canvas}></canvas>
			{#if gameOver}
				<div class="overlay">
					<p class="over-title">GAME OVER</p>
					<p class="over-score">Score: {score}</p>
					<button class="btn-play" onclick={startGame}>Chơi lại</button>
				</div>
			{/if}
			{#if isPaused && !gameOver}
				<div class="overlay">
					<p class="over-title pause-title">TẠM DỪNG</p>
					<p class="over-score">Score: {score}</p>
					<button class="btn-play" onclick={togglePause}>Tiếp tục</button>
				</div>
			{/if}
			{#if !isPlaying && !gameOver}
				<div class="overlay start">
					<p class="over-title">TETRIS</p>
					<button class="btn-play" onclick={startGame}>Bắt đầu</button>
				</div>
			{/if}
		</div>

		<!-- Sidebar -->
		<aside class="sidebar">
			<div class="score-box">
				<span class="label">SCORE</span>
				<span class="value">{score}</span>
			</div>

			<div class="hint-box">
				<p>↑ Xoay</p>
				<p>← → Di chuyển</p>
				<p>↓ Rơi nhanh</p>
				<p>Space Tạm dừng</p>
			</div>

			<div class="btn-group">
				{#if isPlaying && !gameOver}
					<button class="btn-pause full" onclick={togglePause}>
						{isPaused ? '▶ Tiếp tục' : '⏸ Tạm dừng'}
					</button>
				{/if}
				<button class="btn-play full" onclick={startGame}>
					{isPlaying ? 'Restart' : 'Play'}
				</button>
			</div>

			<!-- Mobile controls -->
			<div class="mobile-controls">
				<div class="row">
					<button onclick={rotate}>↑</button>
				</div>
				<div class="row">
					<button onclick={moveLeft}>←</button>
					<button onclick={moveDown}>↓</button>
					<button onclick={moveRight}>→</button>
				</div>
			</div>
		</aside>
	</main>
</div>

<style>
	:global(*, *::before, *::after) { box-sizing: border-box; margin: 0; padding: 0; }

	:global(body) {
		background: #07070f;
		color: #e0e0e0;
		font-family: 'Courier New', Courier, monospace;
		min-height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	/* ── App shell ── */
	.app {
		display: flex;
		flex-direction: column;
		gap: 12px;
		padding: 16px;
		width: fit-content;
	}

	/* ── Header ── */
	header {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.logo {
		font-size: 1.6rem;
		font-weight: 900;
		letter-spacing: 0.25em;
		background: linear-gradient(90deg, #ff4655, #ffe548, #00f5d4);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	.sound-btn {
		background: rgba(255,255,255,0.07);
		border: 1px solid rgba(255,255,255,0.1);
		border-radius: 8px;
		padding: 6px 10px;
		cursor: pointer;
		font-size: 1rem;
		transition: background 0.2s;
	}
	.sound-btn:hover { background: rgba(255,255,255,0.14); }
	.sound-btn.muted { opacity: 0.45; }

	/* ── Main layout ── */
	main {
		display: flex;
		gap: 16px;
		align-items: flex-start;
	}

	/* ── Board ── */
	.board-wrap {
		position: relative;
		border: 2px solid rgba(255,255,255,0.08);
		border-radius: 4px;
		overflow: hidden;
		box-shadow:
			0 0 30px rgba(0,245,212,0.08),
			0 0 60px rgba(72,149,239,0.06);
	}

	canvas {
		display: block;
	}

	/* ── Overlay (start / game over) ── */
	.overlay {
		position: absolute;
		inset: 0;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 16px;
		background: rgba(7,7,15,0.82);
		backdrop-filter: blur(6px);
	}

	.over-title {
		font-size: 2.4rem;
		font-weight: 900;
		letter-spacing: 0.18em;
		background: linear-gradient(90deg, #ff4655, #ffe548);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
		animation: pulse 1.4s ease-in-out infinite;
	}

	.over-score {
		font-size: 1.1rem;
		color: #aaa;
		letter-spacing: 0.08em;
	}

	@keyframes pulse {
		0%, 100% { opacity: 1; }
		50%       { opacity: 0.6; }
	}

	/* ── Sidebar ── */
	.sidebar {
		display: flex;
		flex-direction: column;
		gap: 20px;
		min-width: 130px;
	}

	.score-box {
		display: flex;
		flex-direction: column;
		gap: 4px;
		background: rgba(255,255,255,0.04);
		border: 1px solid rgba(255,255,255,0.07);
		border-radius: 10px;
		padding: 14px 18px;
	}

	.label {
		font-size: 0.6rem;
		letter-spacing: 0.3em;
		color: #666;
	}

	.value {
		font-size: 2rem;
		font-weight: 900;
		color: #ffe548;
		text-shadow: 0 0 12px rgba(255,229,72,0.5);
	}

	.hint-box {
		display: flex;
		flex-direction: column;
		gap: 6px;
		font-size: 0.72rem;
		color: #555;
		line-height: 1.6;
		padding: 0 2px;
	}

	.pause-title {
		background: linear-gradient(90deg, #00f5d4, #4895ef);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	/* ── Button group ── */
	.btn-group {
		display: flex;
		flex-direction: column;
		gap: 8px;
	}

	.btn-pause {
		background: rgba(255,255,255,0.07);
		border: 1px solid rgba(255,255,255,0.15);
		border-radius: 10px;
		padding: 10px 24px;
		font-family: inherit;
		font-size: 0.9rem;
		font-weight: 700;
		letter-spacing: 0.06em;
		color: #ddd;
		cursor: pointer;
		transition: background 0.2s, transform 0.1s;
	}
	.btn-pause:hover  { background: rgba(255,255,255,0.13); }
	.btn-pause:active { transform: scale(0.97); }
	.btn-pause.full   { width: 100%; }

	/* ── Buttons ── */
	.btn-play {
		background: linear-gradient(135deg, #00f5d4, #4895ef);
		border: none;
		border-radius: 10px;
		padding: 12px 24px;
		font-family: inherit;
		font-size: 1rem;
		font-weight: 700;
		letter-spacing: 0.08em;
		color: #07070f;
		cursor: pointer;
		transition: opacity 0.2s, transform 0.1s;
		box-shadow: 0 0 20px rgba(0,245,212,0.3);
	}
	.btn-play:hover  { opacity: 0.88; }
	.btn-play:active { transform: scale(0.97); }
	.btn-play.full   { width: 100%; }

	/* ── Mobile controls ── */
	.mobile-controls {
		display: none;
		flex-direction: column;
		align-items: center;
		gap: 8px;
	}

	.mobile-controls .row {
		display: flex;
		gap: 8px;
	}

	.mobile-controls button {
		width: 52px;
		height: 52px;
		font-size: 1.3rem;
		border-radius: 10px;
		border: 1px solid rgba(255,255,255,0.12);
		background: rgba(255,255,255,0.06);
		color: #ddd;
		cursor: pointer;
		transition: background 0.15s;
		-webkit-tap-highlight-color: transparent;
		user-select: none;
	}
	.mobile-controls button:active { background: rgba(255,255,255,0.18); }

	/* ── Responsive ── */
	@media (max-width: 540px) {
		main { flex-direction: column; align-items: center; }

		canvas {
			width: min(100vw - 24px, 300px) !important;
			height: auto !important;
		}

		.sidebar { width: 100%; flex-direction: row; flex-wrap: wrap; }
		.score-box { flex: 1; }
		.hint-box  { display: none; }
		.btn-group { flex: 1; flex-direction: row; flex-wrap: wrap; }
		.btn-play.full, .btn-pause.full { flex: 1; }

		.mobile-controls { display: flex; width: 100%; justify-content: center; }
	}
</style>