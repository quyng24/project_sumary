<svelte:options runes={true} />
<script lang="ts">
	import { onMount } from 'svelte';
	import BackHome from '$lib/components/BackHome.svelte';

	type Position = { x: number; y: number };
	type PlatformData = { x: number; y: number; width: number; height: number };

	const gravity = 0.65;
	const moveSpeed = 7;
	const jumpForce = 15;
	const playerStart = { x: 110, y: 120 };

	const COLORS = {
		player: '#4ECDC4',
		platformTop: '#ff8c82',
		platformBottom: '#ee5253',
		checkpoint: '#FFE66D',
		checkpointGlow: 'rgba(255, 230, 109, 0.45)',
		bg: '#232946'
	};

	const LEVEL_PLATFORMS: PlatformData[] = [
		{ x: -240, y: 620, width: 880, height: 40 },
		{ x: 410, y: 520, width: 220, height: 24 },
		{ x: 730, y: 445, width: 220, height: 24 },
		{ x: 1040, y: 375, width: 190, height: 24 },
		{ x: 1320, y: 470, width: 240, height: 24 },
		{ x: 1660, y: 395, width: 220, height: 24 },
		{ x: 1980, y: 320, width: 220, height: 24 },
		{ x: 2280, y: 430, width: 280, height: 24 }
	];

	const LEVEL_CHECKPOINTS = [
		{ x: 805, y: 355, message: 'Checkpoint 1: good jump.' },
		{ x: 1735, y: 305, message: 'Checkpoint 2: almost there.' },
		{ x: 2360, y: 340, message: 'Finish line reached.' }
	];

	let canvas: HTMLCanvasElement | undefined;
	let ctx: CanvasRenderingContext2D | null = null;
	let gameStarted = $state(false);
	let checkpointVisible = $state(false);
	let checkpointMessage = $state('');

	let platforms: Platform[] = [];
	let checkpoints: Checkpoint[] = [];
	let scrollOffset = 0;
	let animationFrame = 0;
	let hideCheckpointTimer: ReturnType<typeof setTimeout> | null = null;

	const keys = {
		left: false,
		right: false
	};

	class Player {
		position: Position = { ...playerStart };
		velocity = { x: 0, y: 0 };
		width = 40;
		height = 40;
		isJumping = false;

		reset() {
			this.position = { ...playerStart };
			this.velocity = { x: 0, y: 0 };
			this.isJumping = false;
		}

		draw() {
			if (!ctx) return;

			ctx.shadowBlur = 15;
			ctx.shadowColor = COLORS.player;
			ctx.fillStyle = COLORS.player;
			ctx.beginPath();
			ctx.roundRect(this.position.x, this.position.y, this.width, this.height, 8);
			ctx.fill();
			ctx.shadowBlur = 0;

			ctx.fillStyle = '#fff';
			const eyeOffset = this.velocity.x >= 0 ? 24 : 6;
			ctx.fillRect(this.position.x + eyeOffset, this.position.y + 10, 8, 8);
		}

		update(worldFloor: number) {
			this.position.x += this.velocity.x;
			this.position.y += this.velocity.y;

			if (this.position.y + this.height < worldFloor) {
				this.velocity.y += gravity;
			} else {
				this.position.y = worldFloor - this.height;
				this.velocity.y = 0;
				this.isJumping = false;
			}

			if (this.position.x < 0) {
				this.position.x = 0;
			}
		}
	}

	class Platform {
		position: Position;
		width: number;
		height: number;

		constructor({ x, y, width, height }: PlatformData) {
			this.position = { x, y };
			this.width = width;
			this.height = height;
		}

		draw() {
			if (!ctx) return;

			const gradient = ctx.createLinearGradient(
				this.position.x,
				this.position.y,
				this.position.x,
				this.position.y + this.height
			);
			gradient.addColorStop(0, COLORS.platformTop);
			gradient.addColorStop(1, COLORS.platformBottom);

			ctx.shadowBlur = 14;
			ctx.shadowColor = 'rgba(238, 82, 83, 0.25)';
			ctx.fillStyle = gradient;
			ctx.beginPath();
			ctx.roundRect(this.position.x, this.position.y, this.width, this.height, 8);
			ctx.fill();
			ctx.shadowBlur = 0;
		}
	}

	class Checkpoint {
		position: Position;
		width = 28;
		height = 58;
		message: string;
		reached: boolean;

		constructor({ x, y, message }: { x: number; y: number; message: string }) {
			this.position = { x, y };
			this.message = message;
			this.reached = false;
		}

		draw() {
			if (!ctx) return;

			ctx.fillStyle = '#d9dde8';
			ctx.fillRect(this.position.x, this.position.y, 4, this.height);

			ctx.shadowBlur = 18;
			ctx.shadowColor = COLORS.checkpointGlow;
			ctx.fillStyle = this.reached ? '#81f7c7' : COLORS.checkpoint;
			ctx.beginPath();
			ctx.moveTo(this.position.x + 4, this.position.y + 4);
			ctx.lineTo(this.position.x + this.width, this.position.y + 15);
			ctx.lineTo(this.position.x + 4, this.position.y + 26);
			ctx.closePath();
			ctx.fill();
			ctx.shadowBlur = 0;
		}
	}

	const player = new Player();

	function showCheckpoint(message: string) {
		checkpointMessage = message;
		checkpointVisible = true;

		if (hideCheckpointTimer) {
			clearTimeout(hideCheckpointTimer);
		}

		hideCheckpointTimer = setTimeout(() => {
			checkpointVisible = false;
		}, 1800);
	}

	function initGame() {
		platforms = LEVEL_PLATFORMS.map((platform) => new Platform(platform));
		checkpoints = LEVEL_CHECKPOINTS.map((checkpoint) => new Checkpoint(checkpoint));
		scrollOffset = 0;
		checkpointVisible = false;
		checkpointMessage = '';
		player.reset();
	}

	function resizeCanvas() {
		if (!canvas) return;

		canvas.width = window.innerWidth;
		canvas.height = window.innerHeight;
	}

	function drawBackground() {
		if (!ctx || !canvas) return;

		const sky = ctx.createLinearGradient(0, 0, 0, canvas.height);
		sky.addColorStop(0, '#2c315d');
		sky.addColorStop(1, '#171b38');
		ctx.fillStyle = sky;
		ctx.fillRect(0, 0, canvas.width, canvas.height);

		for (let i = 0; i < 30; i += 1) {
			const x = (((i * 147 - scrollOffset * 0.25) % (canvas.width + 160)) + canvas.width + 160) % (canvas.width + 160);
			const y = 35 + ((i * 59) % Math.max(canvas.height * 0.55, 220));
			ctx.fillStyle = 'rgba(255, 255, 255, 0.55)';
			ctx.beginPath();
			ctx.arc(x - 80, y, 1.8 + (i % 3), 0, Math.PI * 2);
			ctx.fill();
		}

		ctx.fillStyle = 'rgba(255, 255, 255, 0.08)';
		ctx.beginPath();
		ctx.arc(canvas.width - 180, 110, 52, 0, Math.PI * 2);
		ctx.fill();
	}

	function handleWorldScroll() {
		if (!canvas) return;

		const maxScroll = Math.max(
			0,
			LEVEL_PLATFORMS[LEVEL_PLATFORMS.length - 1].x +
				LEVEL_PLATFORMS[LEVEL_PLATFORMS.length - 1].width -
				canvas.width +
				180
		);
		const rightAnchor = Math.min(canvas.width * 0.42, 420);
		const leftAnchor = 110;

		if (keys.right) {
			if (player.position.x < rightAnchor || scrollOffset >= maxScroll) {
				player.velocity.x = moveSpeed;
			} else {
				player.velocity.x = 0;
				scrollOffset = Math.min(scrollOffset + moveSpeed, maxScroll);
				platforms.forEach((platform) => (platform.position.x -= moveSpeed));
				checkpoints.forEach((checkpoint) => (checkpoint.position.x -= moveSpeed));
			}
		} else if (keys.left) {
			if (player.position.x > leftAnchor || scrollOffset <= 0) {
				player.velocity.x = -moveSpeed;
			} else {
				player.velocity.x = 0;
				scrollOffset = Math.max(scrollOffset - moveSpeed, 0);
				platforms.forEach((platform) => (platform.position.x += moveSpeed));
				checkpoints.forEach((checkpoint) => (checkpoint.position.x += moveSpeed));
			}
		} else {
			player.velocity.x = 0;
		}
	}

	function resolvePlatformCollisions(previousY: number) {
		for (const platform of platforms) {
			const isLanding =
				player.velocity.y >= 0 &&
				previousY + player.height <= platform.position.y &&
				player.position.y + player.height >= platform.position.y &&
				player.position.x + player.width >= platform.position.x &&
				player.position.x <= platform.position.x + platform.width;

			if (isLanding) {
				player.position.y = platform.position.y - player.height;
				player.velocity.y = 0;
				player.isJumping = false;
			}
		}
	}

	function checkCheckpointCollisions() {
		for (const checkpoint of checkpoints) {
			const isTouching =
				player.position.x + player.width >= checkpoint.position.x &&
				player.position.x <= checkpoint.position.x + checkpoint.width &&
				player.position.y + player.height >= checkpoint.position.y &&
				player.position.y <= checkpoint.position.y + checkpoint.height;

			if (isTouching && !checkpoint.reached) {
				checkpoint.reached = true;
				showCheckpoint(checkpoint.message);
			}
		}
	}

	function drawScene() {
		if (!ctx) return;

		platforms.forEach((platform) => platform.draw());
		checkpoints.forEach((checkpoint) => checkpoint.draw());
		player.draw();
	}

	function animate() {
		if (!gameStarted || !canvas || !ctx) return;

		animationFrame = requestAnimationFrame(animate);
		drawBackground();
		handleWorldScroll();

		const previousY = player.position.y;
		const worldFloor = canvas.height + 160;

		player.update(worldFloor);
		resolvePlatformCollisions(previousY);
		checkCheckpointCollisions();

		if (player.position.y > canvas.height + 120) {
			showCheckpoint('You fell. Restarting level.');
			initGame();
			return;
		}

		drawScene();
	}

	function startGame() {
		if (!canvas) return;

		cancelAnimationFrame(animationFrame);
		initGame();
		gameStarted = true;
		animate();
	}

	function handleKeydown(event: KeyboardEvent) {
		if (!gameStarted) return;

		if (event.key === 'ArrowRight' || event.key.toLowerCase() === 'd') {
			keys.right = true;
		}

		if (event.key === 'ArrowLeft' || event.key.toLowerCase() === 'a') {
			keys.left = true;
		}

		if ((event.key === ' ' || event.key === 'ArrowUp' || event.key.toLowerCase() === 'w') && !player.isJumping) {
			event.preventDefault();
			player.velocity.y = -jumpForce;
			player.isJumping = true;
		}
	}

	function handleKeyup(event: KeyboardEvent) {
		if (event.key === 'ArrowRight' || event.key.toLowerCase() === 'd') {
			keys.right = false;
		}

		if (event.key === 'ArrowLeft' || event.key.toLowerCase() === 'a') {
			keys.left = false;
		}
	}

	function handleBlur() {
		keys.left = false;
		keys.right = false;
		player.velocity.x = 0;
	}

	onMount(() => {
		if (!canvas) return;

		ctx = canvas.getContext('2d');
		resizeCanvas();
		initGame();
		drawBackground();
		drawScene();

		window.addEventListener('keydown', handleKeydown);
		window.addEventListener('keyup', handleKeyup);
		window.addEventListener('blur', handleBlur);
		window.addEventListener('resize', resizeCanvas);

		return () => {
			cancelAnimationFrame(animationFrame);
			window.removeEventListener('keydown', handleKeydown);
			window.removeEventListener('keyup', handleKeyup);
			window.removeEventListener('blur', handleBlur);
			window.removeEventListener('resize', resizeCanvas);

			if (hideCheckpointTimer) {
				clearTimeout(hideCheckpointTimer);
			}
		};
	});
</script>

<BackHome />

{#if !gameStarted}
	<div class="start-screen">
		<h1 class="main-title">Nguyen Quy Coder</h1>

		<p class="instructions">Help the main player navigate through the platforms.</p>
		<p class="instructions">Use `Arrow Left` / `Arrow Right` or `A` / `D` to move.</p>
		<p class="instructions">Press `Space`, `Arrow Up`, or `W` to jump.</p>

		<div class="btn-container">
			<button class="btn" onclick={startGame}>Start Game</button>
		</div>
	</div>
{/if}

{#if checkpointVisible}
	<div class="checkpoint-screen">
		<h2>Checkpoint</h2>
		<p>{checkpointMessage}</p>
	</div>
{/if}

<canvas bind:this={canvas} class:visible={gameStarted}></canvas>

<style>
	.start-screen {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		width: min(90vw, 500px);
		padding: 40px;
		background: rgba(255, 255, 255, 0.05);
		backdrop-filter: blur(20px);
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 30px;
		text-align: center;
		box-shadow: 0 25px 50px rgba(0, 0, 0, 0.3);
		z-index: 10;
	}

	.main-title {
		margin-bottom: 20px;
		font-size: 2.5rem;
		font-weight: 900;
		background: linear-gradient(to right, #4ECDC4, #FFE66D);
		background-clip: text;
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	.instructions {
		margin-bottom: 10px;
		color: #b8c1ec;
		font-size: 0.95rem;
		line-height: 1.6;
	}

	.btn {
		margin-top: 30px;
		padding: 15px 40px;
		border: none;
		border-radius: 999px;
		background: #ee7c7c;
		color: white;
		font-size: 1.1rem;
		font-weight: bold;
		cursor: pointer;
		transition:
			transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275),
			box-shadow 0.3s ease,
			background 0.3s ease;
		box-shadow: 0 10px 20px rgba(238, 124, 124, 0.3);
	}

	.btn:hover {
		transform: scale(1.05) translateY(-3px);
		box-shadow: 0 15px 25px rgba(238, 124, 124, 0.4);
		background: #f08a8a;
	}

	.checkpoint-screen {
		position: absolute;
		top: 50px;
		left: 50%;
		z-index: 12;
		padding: 15px 30px;
		border-radius: 15px;
		background: rgba(78, 205, 196, 0.94);
		color: #232946;
		font-weight: bold;
		text-align: center;
		transform: translateX(-50%);
		animation: slideDown 0.45s ease;
	}

	.checkpoint-screen h2,
	.checkpoint-screen p {
		margin: 0;
	}

	.checkpoint-screen p {
		margin-top: 0.3rem;
	}

	@keyframes slideDown {
		from {
			transform: translate(-50%, -100px);
		}

		to {
			transform: translate(-50%, 0);
		}
	}

	canvas {
		display: block;
		width: 100vw;
		height: 100vh;
		background: radial-gradient(circle at center, #2d345a 0%, #232946 100%);
		opacity: 0;
		transition: opacity 0.25s ease;
	}

	canvas.visible {
		opacity: 1;
	}
</style>
