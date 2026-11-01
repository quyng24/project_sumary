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
		'#1a1a2e'
	];

	const GLOW_MAPPING: string[] = [
		'rgba(255,70,85,0.6)',
		'rgba(255,159,28,0.6)',
		'rgba(46,196,182,0.6)',
		'rgba(199,125,255,0.6)',
		'rgba(72,149,239,0.6)',
		'rgba(0,245,212,0.6)',
		'rgba(255,229,72,0.6)',
		'transparent'
	];

	const BRICK_LAYOUT: number[][][][] = [
		[
			[[1, 7, 7], [1, 1, 1], [7, 7, 7]],
			[[7, 1, 1], [7, 1, 7], [7, 1, 7]],
			[[7, 7, 7], [1, 1, 1], [7, 7, 1]],
			[[7, 1, 7], [7, 1, 7], [1, 1, 7]]
		],
		[
			[[7, 1, 7], [7, 1, 7], [7, 1, 1]],
			[[7, 7, 7], [1, 1, 1], [1, 7, 7]],
			[[1, 1, 7], [7, 1, 7], [7, 1, 7]],
			[[7, 7, 1], [1, 1, 1], [7, 7, 7]]
		],
		[
			[[1, 7, 7], [1, 1, 7], [7, 1, 7]],
			[[7, 1, 1], [1, 1, 7], [7, 7, 7]],
			[[7, 1, 7], [7, 1, 1], [7, 7, 1]],
			[[7, 7, 7], [7, 1, 1], [1, 1, 7]]
		],
		[
			[[7, 1, 7], [1, 1, 7], [1, 7, 7]],
			[[1, 1, 7], [7, 1, 1], [7, 7, 7]],
			[[7, 7, 1], [7, 1, 1], [7, 1, 7]],
			[[7, 7, 7], [1, 1, 7], [7, 1, 1]]
		],
		[
			[[7, 7, 7, 7], [1, 1, 1, 1], [7, 7, 7, 7], [7, 7, 7, 7]],
			[[7, 7, 1, 7], [7, 7, 1, 7], [7, 7, 1, 7], [7, 7, 1, 7]],
			[[7, 7, 7, 7], [7, 7, 7, 7], [1, 1, 1, 1], [7, 7, 7, 7]],
			[[7, 1, 7, 7], [7, 1, 7, 7], [7, 1, 7, 7], [7, 1, 7, 7]]
		],
		[
			[[7, 7, 7, 7], [7, 1, 1, 7], [7, 1, 1, 7], [7, 7, 7, 7]],
			[[7, 7, 7, 7], [7, 1, 1, 7], [7, 1, 1, 7], [7, 7, 7, 7]],
			[[7, 7, 7, 7], [7, 1, 1, 7], [7, 1, 1, 7], [7, 7, 7, 7]],
			[[7, 7, 7, 7], [7, 1, 1, 7], [7, 1, 1, 7], [7, 7, 7, 7]]
		],
		[
			[[7, 1, 7], [1, 1, 1], [7, 7, 7]],
			[[7, 1, 7], [7, 1, 1], [7, 1, 7]],
			[[7, 7, 7], [1, 1, 1], [7, 1, 7]],
			[[7, 1, 7], [1, 1, 7], [7, 1, 7]]
		]
	];

	type Grid = number[][];

	let canvas = $state<HTMLCanvasElement | null>(null);
	let ctx = $state<CanvasRenderingContext2D | null>(null);

	let grid = $state<Grid>(generateEmptyGrid());
	let score = $state(0);
	let gameOver = $state(false);
	let isPlaying = $state(false);
	let isPaused = $state(false);
	let soundOn = $state(true);

	let brickId = $state(0);
	let activeIndex = $state(0);
	let colPos = $state(3);
	let rowPos = $state(-2);

	let intervalId: ReturnType<typeof setInterval> | null = null;
	let audioCtx: AudioContext | null = null;

	function getAudioContext(): AudioContext | null {
		if (typeof window === 'undefined') return null;
		if (!audioCtx) {
			const AudioContextClass =
				window.AudioContext ||
				(window as unknown as { webkitAudioContext: typeof AudioContext }).webkitAudioContext;
			if (AudioContextClass) audioCtx = new AudioContextClass();
		}
		if (audioCtx && audioCtx.state === 'suspended') {
			audioCtx.resume();
		}
		return audioCtx;
	}

	function generateEmptyGrid(): Grid {
		return Array.from({ length: ROWS }, () => Array(COLS).fill(WHITE_COLOR_ID));
	}

	function drawCell(x: number, y: number, colorId: number, ghost = false) {
		if (!ctx) return;
		const color = COLOR_MAPPING[colorId] ?? COLOR_MAPPING[WHITE_COLOR_ID];
		const glow = GLOW_MAPPING[colorId] ?? 'transparent';
		const px = x * BLOCK_SIZE;
		const py = y * BLOCK_SIZE;
		const bs = BLOCK_SIZE;

		if (colorId === WHITE_COLOR_ID) {
			ctx.fillStyle = '#0d0d1a';
			ctx.fillRect(px, py, bs, bs);
			ctx.strokeStyle = 'rgba(255,255,255,0.035)';
			ctx.lineWidth = 0.5;
			ctx.strokeRect(px + 0.25, py + 0.25, bs - 0.5, bs - 0.5);
			return;
		}

		if (ghost) {
			ctx.fillStyle = color + '28';
			ctx.fillRect(px + 1, py + 1, bs - 2, bs - 2);
			ctx.strokeStyle = color + '65';
			ctx.lineWidth = 1;
			ctx.strokeRect(px + 1, py + 1, bs - 2, bs - 2);
			return;
		}

		const grad = ctx.createLinearGradient(px, py, px + bs, py + bs);
		grad.addColorStop(0, lighten(color, 40));
		grad.addColorStop(1, color);
		ctx.fillStyle = grad;
		ctx.fillRect(px + 1, py + 1, bs - 2, bs - 2);

		ctx.shadowColor = glow;
		ctx.shadowBlur = 10;
		ctx.fillRect(px + 1, py + 1, bs - 2, bs - 2);
		ctx.shadowBlur = 0;

		ctx.fillStyle = 'rgba(255,255,255,0.18)';
		ctx.fillRect(px + 2, py + 2, bs - 4, 3.5);
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
		rid: number,
		aidx: number,
		cpos: number,
		rpos: number,
		colorId: number,
		ghost = false
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
					)
						return true;
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

	function hardDrop() {
		if (!isPlaying || gameOver || isPaused) return;
		const ghost = getGhostRow();
		rowPos = ghost;
		renderFrame();
		handleLanded();
		if (soundOn) playBeep(520, 0.1);
	}

	function handleLanded() {
		if (rowPos <= 0) {
			handleGameOver();
			return;
		}
		const layout = getLayout();
		const newGrid = grid.map((row) => [...row]);
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
		const remaining = grid.filter((row) => row.some((cell) => cell === WHITE_COLOR_ID));
		const cleared = ROWS - remaining.length;
		if (cleared > 0) {
			const newRows = Array.from({ length: cleared }, () => Array(COLS).fill(WHITE_COLOR_ID));
			grid = [...newRows, ...remaining];
			score += cleared * 10;
			if (soundOn) playBeep(880, 0.2);
		}
	}

	function generateNewBrick() {
		brickId = Math.floor(Math.random() * BRICK_LAYOUT.length);
		activeIndex = 0;
		colPos = 3;
		rowPos = -2;
	}

	function handleGameOver() {
		gameOver = true;
		isPlaying = false;
		if (intervalId) clearInterval(intervalId);
	}

	function playBeep(freq = 880, duration = 0.15) {
		if (!soundOn) return;
		try {
			const ctxAudio = getAudioContext();
			if (!ctxAudio) return;
			const osc = ctxAudio.createOscillator();
			const gain = ctxAudio.createGain();
			osc.connect(gain);
			gain.connect(ctxAudio.destination);
			osc.type = 'sine';
			osc.frequency.setValueAtTime(freq, ctxAudio.currentTime);
			osc.frequency.exponentialRampToValueAtTime(freq * 0.5, ctxAudio.currentTime + duration);
			gain.gain.setValueAtTime(0.2, ctxAudio.currentTime);
			gain.gain.exponentialRampToValueAtTime(0.001, ctxAudio.currentTime + duration);
			osc.start();
			osc.stop(ctxAudio.currentTime + duration);
		} catch (_) {}
	}

	function togglePause() {
		if (!isPlaying || gameOver) return;
		isPaused = !isPaused;
	}

	function startGame() {
		if (intervalId) clearInterval(intervalId);
		grid = generateEmptyGrid();
		score = 0;
		gameOver = false;
		isPlaying = true;
		isPaused = false;
		generateNewBrick();
		renderFrame();
		intervalId = setInterval(() => {
			if (!gameOver) moveDown();
			else if (intervalId) clearInterval(intervalId);
		}, 500);
	}

	function handleKeyDown(e: KeyboardEvent) {
		if (e.code === 'KeyP') {
			e.preventDefault();
			togglePause();
			return;
		}
		if (e.code === 'Space' && isPlaying && !gameOver) {
			e.preventDefault();
			hardDrop();
			return;
		}
		if (!isPlaying || gameOver || isPaused) return;
		switch (e.code) {
			case 'ArrowLeft':
			case 'KeyA':
				e.preventDefault();
				moveLeft();
				break;
			case 'ArrowRight':
			case 'KeyD':
				e.preventDefault();
				moveRight();
				break;
			case 'ArrowDown':
			case 'KeyS':
				e.preventDefault();
				moveDown();
				break;
			case 'ArrowUp':
			case 'KeyW':
				e.preventDefault();
				rotate();
				break;
		}
	}

	onMount(() => {
		if (!canvas) return;
		ctx = canvas.getContext('2d');
		canvas.width = COLS * BLOCK_SIZE;
		canvas.height = ROWS * BLOCK_SIZE;
		drawBoard();

		window.addEventListener('keydown', handleKeyDown);
		return () => {
			window.removeEventListener('keydown', handleKeyDown);
			if (intervalId) clearInterval(intervalId);
		};
	});
</script>

<svelte:head>
	<title>Tetris Cyber Arcade | Game Xếp Gạch Siêu Mượt</title>
	<meta
		name="viewport"
		content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover"
	/>
	<meta
		name="description"
		content="Chơi game xếp hình Tetris kinh điển giao diện Cyberpunk neon, tối ưu hoàn hảo cho màn hình di động 100vh không cần cuộn."
	/>
</svelte:head>

<div class="tetris-page">
	<!-- Thanh điều hướng gọn gàng trên cùng -->
	<header class="top-nav">
		<div class="nav-left">
			<BackHome label="Trang chủ" />
		</div>

		<div class="nav-center">
			<span class="logo">TETRIS</span>
			<div class="score-chip">
				<span class="chip-label">SCORE</span>
				<span class="chip-val">{score}</span>
			</div>
		</div>

		<div class="nav-right">
			<button
				type="button"
				class="icon-btn"
				class:muted={!soundOn}
				onclick={() => (soundOn = !soundOn)}
				title={soundOn ? 'Tắt âm thanh' : 'Bật âm thanh'}
				aria-label={soundOn ? 'Tắt âm thanh' : 'Bật âm thanh'}
			>
				{soundOn ? '🔊' : '🔇'}
			</button>

			{#if isPlaying && !gameOver}
				<button
					type="button"
					class="icon-btn pause-toggle"
					onclick={togglePause}
					title={isPaused ? 'Tiếp tục' : 'Tạm dừng'}
					aria-label={isPaused ? 'Tiếp tục' : 'Tạm dừng'}
				>
					{isPaused ? '▶' : '⏸'}
				</button>
			{/if}
		</div>
	</header>

	<!-- Khu vực chơi chính -->
	<div class="game-stage">
		<main class="main-content">
			<!-- Khung bảng game Canvas -->
			<div class="board-wrap">
				<canvas bind:this={canvas}></canvas>

				{#if gameOver}
					<div class="overlay">
						<p class="over-title">GAME OVER</p>
						<p class="over-score">Score: {score}</p>
						<button type="button" class="btn-play" onclick={startGame}>Chơi lại</button>
					</div>
				{/if}

				{#if isPaused && !gameOver}
					<div class="overlay">
						<p class="over-title pause-title">TẠM DỪNG</p>
						<p class="over-score">Score: {score}</p>
						<button type="button" class="btn-play" onclick={togglePause}>Tiếp tục</button>
					</div>
				{/if}

				{#if !isPlaying && !gameOver}
					<div class="overlay start">
						<p class="over-title">TETRIS</p>
						<p class="start-sub">CYBER ARCADE</p>
						<button type="button" class="btn-play" onclick={startGame}>Bắt đầu chơi</button>
					</div>
				{/if}
			</div>

			<!-- Sidebar chỉ hiển thị trên máy tính (Desktop / Tablet ngang) -->
			<aside class="desktop-sidebar">
				<div class="score-card">
					<span class="label">SCORE</span>
					<span class="value">{score}</span>
				</div>

				<div class="controls-guide">
					<span class="guide-heading">ĐIỀU KHIỂN PHÍM</span>
					<div class="guide-item"><kbd>↑</kbd><span>Xoay khối</span></div>
					<div class="guide-item"><kbd>←</kbd><kbd>→</kbd><span>Di chuyển</span></div>
					<div class="guide-item"><kbd>↓</kbd><span>Rơi nhanh</span></div>
					<div class="guide-item"><kbd>Space</kbd><span>Rơi ngay (Hard drop)</span></div>
					<div class="guide-item"><kbd>P</kbd><span>Tạm dừng</span></div>
				</div>

				<div class="action-buttons">
					{#if isPlaying && !gameOver}
						<button type="button" class="btn-secondary" onclick={togglePause}>
							{isPaused ? '▶ Tiếp tục' : '⏸ Tạm dừng'}
						</button>
					{/if}
					<button type="button" class="btn-play" onclick={startGame}>
						{isPlaying ? 'Khởi động lại' : 'Chơi ngay'}
					</button>
				</div>
			</aside>
		</main>

		<!-- 
			Bàn phím ảo công thái học hai tay trên Mobile (Vừa khít 100vh không cần cuộn):
			- Tay trái: Phím di chuyển Trái / Xuống / Phải
			- Tay phải: Phím Xoay khối & Rơi ngay tức thì (Hard drop)
		-->
		<div class="mobile-controls-deck" aria-label="Bàn phím ảo di động">
			<div class="dpad-group">
				<button type="button" class="dpad-btn btn-left" onclick={moveLeft} aria-label="Sang trái">
					←
				</button>
				<button type="button" class="dpad-btn btn-down" onclick={moveDown} aria-label="Rơi xuống">
					↓
				</button>
				<button type="button" class="dpad-btn btn-right" onclick={moveRight} aria-label="Sang phải">
					→
				</button>
			</div>

			<div class="action-group">
				<button
					type="button"
					class="action-btn btn-harddrop"
					onclick={hardDrop}
					aria-label="Rơi ngay lập tức"
				>
					<span class="action-icon">⚡</span>
					<span class="action-label">RƠI</span>
				</button>
				<button
					type="button"
					class="action-btn btn-rotate"
					onclick={rotate}
					aria-label="Xoay khối"
				>
					<span class="action-icon">↻</span>
					<span class="action-label">XOAY</span>
				</button>
			</div>
		</div>
	</div>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@600;700;800&family=JetBrains+Mono:wght@600;700;900&display=swap');

	/* 
	   Trang chính: Cố định tuyệt đối 100vh / 100dvh, ngăn 100% cuộn trang hoặc nảy trang trên mobile 
	*/
	.tetris-page {
		position: fixed;
		inset: 0;
		width: 100vw;
		height: 100vh;
		height: 100dvh;
		max-height: 100dvh;
		background-color: #070711;
		background-image: 
			radial-gradient(circle at 50% 10%, #151838 0%, transparent 60%),
			radial-gradient(circle at 85% 85%, rgba(0, 245, 212, 0.05) 0%, transparent 40%);
		display: flex;
		flex-direction: column;
		color: #ffffff;
		font-family: 'Plus Jakarta Sans', sans-serif;
		overflow: hidden;
		user-select: none;
		-webkit-user-select: none;
		touch-action: manipulation;
		overscroll-behavior: none;
		box-sizing: border-box;
	}

	/* =================================================
	   TOP NAV (Chiều cao cố định 48px, không chiếm diện tích)
	   ================================================= */
	.top-nav {
		width: 100%;
		height: 48px;
		min-height: 48px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 1rem;
		background: rgba(13, 13, 26, 0.75);
		border-bottom: 1px solid rgba(255, 255, 255, 0.08);
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
		z-index: 50;
		box-sizing: border-box;
		flex-shrink: 0;
	}

	.nav-left {
		display: flex;
		align-items: center;
	}

	:global(.top-nav .back-home) {
		position: static !important;
		top: auto !important;
		left: auto !important;
		display: inline-flex !important;
		padding: 0.35rem 0.8rem !important;
		font-size: 0.8rem !important;
		border-radius: 999px !important;
		background: rgba(255, 255, 255, 0.06) !important;
		border: 1px solid rgba(255, 255, 255, 0.12) !important;
		color: #ffffff !important;
	}

	.nav-center {
		display: flex;
		align-items: center;
		gap: 0.75rem;
	}

	.logo {
		font-family: 'JetBrains Mono', monospace;
		font-size: 1.25rem;
		font-weight: 900;
		letter-spacing: 0.18em;
		background: linear-gradient(90deg, #ff4655, #ffe548, #00f5d4);
		-webkit-background-clip: text;
		background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	/* Chip hiển thị điểm số trên thanh header */
	.score-chip {
		display: flex;
		align-items: baseline;
		gap: 0.35rem;
		padding: 0.2rem 0.65rem;
		border-radius: 999px;
		background: rgba(255, 229, 72, 0.12);
		border: 1px solid rgba(255, 229, 72, 0.28);
	}

	.chip-label {
		font-size: 0.62rem;
		font-weight: 700;
		color: #94a3b8;
		letter-spacing: 0.05em;
	}

	.chip-val {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.85rem;
		font-weight: 900;
		color: #ffe548;
	}

	.nav-right {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	.icon-btn {
		width: 34px;
		height: 34px;
		display: flex;
		align-items: center;
		justify-content: center;
		background: rgba(255, 255, 255, 0.06);
		border: 1px solid rgba(255, 255, 255, 0.12);
		border-radius: 8px;
		color: #f8fafc;
		font-size: 0.95rem;
		cursor: pointer;
		transition: all 0.2s ease;
		touch-action: manipulation;
	}

	.icon-btn:active {
		transform: scale(0.92);
		background: rgba(255, 255, 255, 0.15);
	}

	.icon-btn.muted {
		opacity: 0.5;
	}

	/* =================================================
	   GAME STAGE & RESPONSIVE CONTAINER
	   ================================================= */
	.game-stage {
		flex: 1;
		min-height: 0;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: space-between;
		padding: 0.5rem 1rem 0.5rem;
		box-sizing: border-box;
		overflow: hidden;
	}

	.main-content {
		flex: 1;
		min-height: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 1.75rem;
		width: 100%;
		max-width: 720px;
	}

	/* =================================================
	   BẢNG GAME CANVAS (Tự động co giãn theo 100vh)
	   ================================================= */
	.board-wrap {
		position: relative;
		height: 100%;
		max-height: calc(100dvh - 48px - 105px - 2rem);
		aspect-ratio: 10 / 20;
		width: auto;
		border: 3px solid #1a1a36;
		border-radius: 8px;
		overflow: hidden;
		box-shadow: 0 0 25px rgba(0, 0, 0, 0.7), 0 0 15px rgba(0, 245, 212, 0.12);
		background: #0d0d1a;
		flex-shrink: 1;
	}

	canvas {
		width: 100%;
		height: 100%;
		display: block;
		background: #0d0d1a;
	}

	/* Overlay Tạm dừng / Game Over / Bắt đầu */
	.overlay {
		position: absolute;
		inset: 0;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 0.85rem;
		background: rgba(7, 7, 17, 0.88);
		backdrop-filter: blur(10px);
		-webkit-backdrop-filter: blur(10px);
		z-index: 20;
		padding: 1rem;
		text-align: center;
	}

	.over-title {
		font-family: 'JetBrains Mono', monospace;
		font-size: clamp(1.4rem, 4vw, 2.2rem);
		font-weight: 900;
		letter-spacing: 0.05em;
		background: linear-gradient(90deg, #ff4655, #ffe548);
		-webkit-background-clip: text;
		background-clip: text;
		-webkit-text-fill-color: transparent;
		margin: 0;
	}

	.pause-title {
		background: linear-gradient(90deg, #00f5d4, #4895ef);
		-webkit-background-clip: text;
		background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	.start-sub {
		font-size: 0.75rem;
		letter-spacing: 0.25em;
		color: #94a3b8;
		margin-top: -0.5rem;
		margin-bottom: 0.5rem;
	}

	.over-score {
		font-family: 'JetBrains Mono', monospace;
		font-size: 1.15rem;
		font-weight: 700;
		color: #ffe548;
		margin: 0;
	}

	.btn-play {
		background: linear-gradient(135deg, #00f5d4 0%, #4895ef 100%);
		color: #070711;
		border: none;
		border-radius: 8px;
		padding: 0.75rem 1.6rem;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.88rem;
		font-weight: 800;
		letter-spacing: 0.06em;
		cursor: pointer;
		box-shadow: 0 4px 18px rgba(0, 245, 212, 0.35);
		transition: transform 0.2s ease, filter 0.2s ease;
		touch-action: manipulation;
	}

	.btn-play:hover {
		transform: translateY(-2px);
		filter: brightness(1.1);
	}

	.btn-play:active {
		transform: scale(0.96);
	}

	.btn-secondary {
		background: rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.16);
		color: #ffffff;
		border-radius: 8px;
		padding: 0.7rem 1.2rem;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.82rem;
		font-weight: 700;
		cursor: pointer;
		transition: all 0.2s ease;
		touch-action: manipulation;
	}

	.btn-secondary:hover {
		background: rgba(255, 255, 255, 0.15);
	}

	/* =================================================
	   DESKTOP SIDEBAR
	   ================================================= */
	.desktop-sidebar {
		display: flex;
		flex-direction: column;
		gap: 1.25rem;
		width: 175px;
	}

	.score-card {
		background: #121324;
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 10px;
		padding: 1.1rem;
		text-align: center;
		box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
	}

	.score-card .label {
		font-size: 0.68rem;
		font-weight: 700;
		color: #94a3b8;
		letter-spacing: 0.18em;
		display: block;
		margin-bottom: 0.35rem;
	}

	.score-card .value {
		font-family: 'JetBrains Mono', monospace;
		font-size: 2.1rem;
		font-weight: 900;
		color: #ffe548;
		display: block;
		text-shadow: 0 0 14px rgba(255, 229, 72, 0.4);
	}

	.controls-guide {
		background: rgba(18, 19, 36, 0.6);
		border: 1px solid rgba(255, 255, 255, 0.06);
		border-radius: 10px;
		padding: 1rem;
		display: flex;
		flex-direction: column;
		gap: 0.65rem;
	}

	.guide-heading {
		font-size: 0.65rem;
		font-weight: 700;
		letter-spacing: 0.15em;
		color: #64748b;
		margin-bottom: 0.2rem;
	}

	.guide-item {
		display: flex;
		align-items: center;
		justify-content: space-between;
		font-size: 0.75rem;
		color: #cbd5e1;
	}

	.guide-item kbd {
		background: #1e2038;
		border: 1px solid rgba(255, 255, 255, 0.12);
		border-radius: 4px;
		padding: 0.15rem 0.45rem;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.72rem;
		font-weight: 700;
		color: #00f5d4;
	}

	.action-buttons {
		display: flex;
		flex-direction: column;
		gap: 0.65rem;
	}

	/* =================================================
	   BÀN PHÍM ẢO DI ĐỘNG (MOBILE CONTROLS DECK)
	   - Chiều cao cố định ~88px, vừa khít trong 100vh
	   - Bố cục 2 tay công thái học tiện lợi
	   ================================================= */
	.mobile-controls-deck {
		display: none;
		width: 100%;
		max-width: 380px;
		align-items: center;
		justify-content: space-between;
		gap: 0.75rem;
		padding: 0.25rem 0.5rem 0.4rem;
		box-sizing: border-box;
		flex-shrink: 0;
	}

	.dpad-group {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	.dpad-btn {
		width: 48px;
		height: 48px;
		background: #151833;
		border: 1px solid rgba(255, 255, 255, 0.14);
		border-radius: 10px;
		color: #f8fafc;
		font-size: 1.35rem;
		font-weight: 900;
		display: flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
		touch-action: manipulation;
		-webkit-tap-highlight-color: transparent;
		box-shadow: 0 4px 10px rgba(0, 0, 0, 0.4);
		transition: transform 0.1s ease, background 0.1s ease;
	}

	.dpad-btn:active {
		transform: scale(0.9);
		background: #232752;
		border-color: #00f5d4;
		color: #00f5d4;
	}

	.action-group {
		display: flex;
		align-items: center;
		gap: 0.6rem;
	}

	.action-btn {
		width: 52px;
		height: 48px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		border-radius: 10px;
		cursor: pointer;
		touch-action: manipulation;
		-webkit-tap-highlight-color: transparent;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
		transition: transform 0.1s ease, filter 0.1s ease;
	}

	.action-btn:active {
		transform: scale(0.9);
		filter: brightness(1.2);
	}

	/* Nút Xoay khối (Màu xanh neon nổi bật) */
	.btn-rotate {
		background: linear-gradient(135deg, #00f5d4 0%, #2ec4b6 100%);
		border: 1px solid rgba(0, 245, 212, 0.4);
		color: #06121e;
	}

	/* Nút Rơi ngay Hard Drop (Màu cam hổ phách rực rỡ) */
	.btn-harddrop {
		background: linear-gradient(135deg, #ff9f1c 0%, #ff4655 100%);
		border: 1px solid rgba(255, 159, 28, 0.4);
		color: #ffffff;
	}

	.action-icon {
		font-size: 1.15rem;
		font-weight: 900;
		line-height: 1;
	}

	.action-label {
		font-size: 0.58rem;
		font-weight: 800;
		font-family: 'JetBrains Mono', monospace;
		letter-spacing: 0.05em;
		margin-top: 2px;
	}

	/* =================================================
	   RESPONSIVE (MOBILE VIEWPORT < 768px)
	   ================================================= */
	@media (max-width: 768px) {
		.desktop-sidebar {
			display: none;
		}

		.mobile-controls-deck {
			display: flex;
		}

		.game-stage {
			padding: 0.25rem 0.75rem 0.35rem;
		}

		.main-content {
			justify-content: center;
			flex: 1;
			min-height: 0;
		}

		/* Đảm bảo board-wrap tự co giãn theo chiều cao khả dụng trong 100vh */
		.board-wrap {
			max-height: calc(100dvh - 48px - 64px - 1rem);
			max-width: calc(100vw - 1.5rem);
		}
	}

	@media (max-width: 420px) {
		.top-nav {
			padding: 0 0.65rem;
		}

		.logo {
			font-size: 1.05rem;
		}

		.score-chip {
			padding: 0.15rem 0.45rem;
		}

		.chip-val {
			font-size: 0.78rem;
		}

		.dpad-btn {
			width: 44px;
			height: 44px;
			font-size: 1.2rem;
		}

		.action-btn {
			width: 46px;
			height: 44px;
		}

		.board-wrap {
			max-height: calc(100dvh - 48px - 60px - 0.75rem);
		}
	}
</style>
