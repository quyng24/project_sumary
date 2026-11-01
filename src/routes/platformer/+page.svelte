<svelte:options runes={true} />

<script lang="ts">
	import { onMount } from 'svelte';
	import BackHome from '$lib/components/BackHome.svelte';

	type Position = { x: number; y: number };
	type PlatformData = { x: number; y: number; width: number; height: number };

	// Hằng số vật lý
	const gravity = 0.62;
	const moveSpeed = 6.5;
	const jumpForce = 13.5;
	const playerStart = { x: 110, y: 120 };

	const COLORS = {
		player: '#4ECDC4',
		playerGlow: 'rgba(78, 205, 196, 0.65)',
		platformTop: '#ff8c82',
		platformBottom: '#ee5253',
		checkpoint: '#FFE66D',
		checkpointGlow: 'rgba(255, 230, 109, 0.45)',
		bg: '#232946'
	};

	// Danh sách bệ nhảy
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

	// Danh sách Checkpoint
	const LEVEL_CHECKPOINTS = [
		{ x: 805, y: 355, message: 'Checkpoint 1: Xuất sắc! Vượt qua nửa chặng đầu.' },
		{ x: 1735, y: 305, message: 'Checkpoint 2: Sắp tới đích rồi, cố lên!' },
		{ x: 2360, y: 340, message: 'CHIẾN THẮNG! Bạn đã hoàn thành xuất sắc toàn bộ màn chơi! 🏆' }
	];

	let canvas: HTMLCanvasElement | undefined;
	let ctx: CanvasRenderingContext2D | null = null;
	let gameStarted = $state(false);
	let gameWon = $state(false);
	let checkpointVisible = $state(false);
	let checkpointMessage = $state('');

	// Điểm số, xu & mạng
	let score = $state(0);
	let coinsCollected = $state(0);
	let totalCoins = $state(18);
	let lives = $state(3);

	let platforms: Platform[] = [];
	let movingPlatforms: MovingPlatform[] = [];
	let springPads: SpringPad[] = [];
	let coins: Coin[] = [];
	let enemies: SlimeEnemy[] = [];
	let particles: Particle[] = [];
	let checkpoints: Checkpoint[] = [];

	let scrollOffset = 0;
	let cameraY = 0;
	let animationFrame = 0;
	let hideCheckpointTimer: ReturnType<typeof setTimeout> | null = null;
	let lastRespawn = { x: playerStart.x, y: playerStart.y, scrollOffset: 0 };

	let audioCtx: AudioContext | null = null;

	const keys = {
		left: false,
		right: false
	};

	// ─── HỆ THỐNG ÂM THANH RETRO SYNTH (WEB AUDIO API) ───
	function getAudioCtx(): AudioContext | null {
		if (typeof window === 'undefined') return null;
		if (!audioCtx) {
			const AudioClass =
				window.AudioContext ||
				(window as unknown as { webkitAudioContext: typeof AudioContext }).webkitAudioContext;
			if (AudioClass) audioCtx = new AudioClass();
		}
		if (audioCtx && audioCtx.state === 'suspended') {
			audioCtx.resume();
		}
		return audioCtx;
	}

	function soundJump() {
		try {
			const c = getAudioCtx();
			if (!c) return;
			const osc = c.createOscillator();
			const gain = c.createGain();
			osc.connect(gain);
			gain.connect(c.destination);
			osc.type = 'sine';
			osc.frequency.setValueAtTime(170, c.currentTime);
			osc.frequency.exponentialRampToValueAtTime(360, c.currentTime + 0.12);
			gain.gain.setValueAtTime(0.18, c.currentTime);
			gain.gain.exponentialRampToValueAtTime(0.001, c.currentTime + 0.12);
			osc.start();
			osc.stop(c.currentTime + 0.12);
		} catch (_) {}
	}

	function soundDoubleJump() {
		try {
			const c = getAudioCtx();
			if (!c) return;
			const osc = c.createOscillator();
			const gain = c.createGain();
			osc.connect(gain);
			gain.connect(c.destination);
			osc.type = 'triangle';
			osc.frequency.setValueAtTime(320, c.currentTime);
			osc.frequency.exponentialRampToValueAtTime(680, c.currentTime + 0.15);
			gain.gain.setValueAtTime(0.2, c.currentTime);
			gain.gain.exponentialRampToValueAtTime(0.001, c.currentTime + 0.15);
			osc.start();
			osc.stop(c.currentTime + 0.15);
		} catch (_) {}
	}

	function soundCoin() {
		try {
			const c = getAudioCtx();
			if (!c) return;
			const t = c.currentTime;
			const osc = c.createOscillator();
			const gain = c.createGain();
			osc.connect(gain);
			gain.connect(c.destination);
			osc.type = 'sine';
			osc.frequency.setValueAtTime(987.77, t);
			osc.frequency.setValueAtTime(1318.51, t + 0.08);
			gain.gain.setValueAtTime(0.22, t);
			gain.gain.exponentialRampToValueAtTime(0.001, t + 0.25);
			osc.start();
			osc.stop(t + 0.25);
		} catch (_) {}
	}

	function soundSpring() {
		try {
			const c = getAudioCtx();
			if (!c) return;
			const osc = c.createOscillator();
			const gain = c.createGain();
			osc.connect(gain);
			gain.connect(c.destination);
			osc.type = 'triangle';
			const t = c.currentTime;
			osc.frequency.setValueAtTime(180, t);
			osc.frequency.linearRampToValueAtTime(620, t + 0.12);
			osc.frequency.exponentialRampToValueAtTime(280, t + 0.26);
			gain.gain.setValueAtTime(0.25, t);
			gain.gain.exponentialRampToValueAtTime(0.001, t + 0.26);
			osc.start();
			osc.stop(t + 0.26);
		} catch (_) {}
	}

	function soundStomp() {
		try {
			const c = getAudioCtx();
			if (!c) return;
			const osc = c.createOscillator();
			const gain = c.createGain();
			osc.connect(gain);
			gain.connect(c.destination);
			osc.type = 'square';
			const t = c.currentTime;
			osc.frequency.setValueAtTime(160, t);
			osc.frequency.exponentialRampToValueAtTime(50, t + 0.14);
			gain.gain.setValueAtTime(0.24, t);
			gain.gain.exponentialRampToValueAtTime(0.001, t + 0.16);
			osc.start();
			osc.stop(t + 0.16);
		} catch (_) {}
	}

	function soundCheckpoint() {
		try {
			const c = getAudioCtx();
			if (!c) return;
			const notes = [523.25, 659.25, 783.99, 1046.5];
			notes.forEach((freq, idx) => {
				const osc = c.createOscillator();
				const gain = c.createGain();
				osc.connect(gain);
				gain.connect(c.destination);
				osc.type = 'sine';
				const t = c.currentTime + idx * 0.08;
				osc.frequency.setValueAtTime(freq, t);
				gain.gain.setValueAtTime(0.18, t);
				gain.gain.exponentialRampToValueAtTime(0.001, t + 0.25);
				osc.start(t);
				osc.stop(t + 0.25);
			});
		} catch (_) {}
	}

	function soundVictory() {
		try {
			const c = getAudioCtx();
			if (!c) return;
			const notes = [523.25, 659.25, 783.99, 1046.5, 1318.51];
			notes.forEach((freq, idx) => {
				const osc = c.createOscillator();
				const gain = c.createGain();
				osc.connect(gain);
				gain.connect(c.destination);
				osc.type = 'triangle';
				const t = c.currentTime + idx * 0.1;
				osc.frequency.setValueAtTime(freq, t);
				gain.gain.setValueAtTime(0.25, t);
				gain.gain.exponentialRampToValueAtTime(0.001, t + 0.35);
				osc.start(t);
				osc.stop(t + 0.35);
			});
		} catch (_) {}
	}

	// ─── HỆ THỐNG HẠT PARTICLE ───
	interface Particle {
		x: number;
		y: number;
		vx: number;
		vy: number;
		size: number;
		color: string;
		alpha: number;
		decay: number;
	}

	function spawnDust(x: number, y: number, color = 'rgba(255,255,255,0.7)', count = 5) {
		for (let i = 0; i < count; i++) {
			particles.push({
				x,
				y,
				vx: (Math.random() - 0.5) * 4,
				vy: -Math.random() * 2.5,
				size: Math.random() * 4 + 2,
				color,
				alpha: 1,
				decay: Math.random() * 0.04 + 0.03
			});
		}
	}

	function spawnSparkles(x: number, y: number, color = '#ffd32a', count = 8) {
		for (let i = 0; i < count; i++) {
			const angle = (Math.PI * 2 * i) / count;
			const speed = Math.random() * 3 + 2;
			particles.push({
				x,
				y,
				vx: Math.cos(angle) * speed,
				vy: Math.sin(angle) * speed,
				size: Math.random() * 4 + 3,
				color,
				alpha: 1,
				decay: 0.03
			});
		}
	}

	function updateParticles() {
		for (let i = particles.length - 1; i >= 0; i--) {
			const p = particles[i];
			p.x += p.vx;
			p.y += p.vy;
			p.alpha -= p.decay;
			if (p.alpha <= 0) {
				particles.splice(i, 1);
			}
		}
	}

	function drawParticles() {
		if (!ctx) return;
		for (const p of particles) {
			ctx.save();
			ctx.globalAlpha = Math.max(0, p.alpha);
			ctx.fillStyle = p.color;
			ctx.beginPath();
			ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
			ctx.fill();
			ctx.restore();
		}
	}

	// ─── NHÂN VẬT CHÍNH (PLAYER) ───
	class Player {
		position: Position = { ...playerStart };
		velocity = { x: 0, y: 0 };
		width = 40;
		height = 40;
		jumpCount = 0;
		facing = 1;

		reset() {
			this.position = { ...playerStart };
			this.velocity = { x: 0, y: 0 };
			this.jumpCount = 0;
			this.facing = 1;
		}

		draw() {
			if (!ctx) return;

			ctx.shadowBlur = 16;
			ctx.shadowColor = COLORS.playerGlow;
			ctx.fillStyle = COLORS.player;
			ctx.beginPath();
			ctx.roundRect(this.position.x, this.position.y, this.width, this.height, 8);
			ctx.fill();
			ctx.shadowBlur = 0;

			// Mắt nhân vật linh hoạt theo hướng chạy
			ctx.fillStyle = '#ffffff';
			const eyeOffset = this.facing > 0 ? 24 : 6;
			ctx.fillRect(this.position.x + eyeOffset, this.position.y + 10, 8, 8);
			ctx.fillStyle = '#1e293b';
			ctx.fillRect(this.position.x + eyeOffset + (this.facing > 0 ? 3 : 1), this.position.y + 12, 4, 5);

			// Vạch ánh sáng trán
			ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
			ctx.fillRect(this.position.x + 8, this.position.y + 4, this.width - 16, 3);
		}

		update(worldFloor: number) {
			this.position.x += this.velocity.x;
			this.position.y += this.velocity.y;

			if (this.velocity.x > 0.1) this.facing = 1;
			else if (this.velocity.x < -0.1) this.facing = -1;

			if (this.position.y + this.height < worldFloor) {
				this.velocity.y += gravity;
			} else {
				this.position.y = worldFloor - this.height;
				this.velocity.y = 0;
				this.jumpCount = 0;
			}

			if (this.position.x < 0) {
				this.position.x = 0;
			}
		}
	}

	// ─── BỆ TĨNH ───
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

	// ─── BỆ TRƯỢT DI ĐỘNG (MOVING PLATFORM) ───
	class MovingPlatform {
		position: Position;
		width: number;
		height: number;
		minX: number;
		maxX: number;
		speed: number;

		constructor(x: number, y: number, width: number, height: number, rangeX: number, speed = 2) {
			this.position = { x, y };
			this.width = width;
			this.height = height;
			this.minX = x;
			this.maxX = x + rangeX;
			this.speed = speed;
		}

		update() {
			this.position.x += this.speed;
			if (this.position.x > this.maxX) {
				this.position.x = this.maxX;
				this.speed = -Math.abs(this.speed);
			} else if (this.position.x < this.minX) {
				this.position.x = this.minX;
				this.speed = Math.abs(this.speed);
			}
		}

		draw() {
			if (!ctx) return;

			const gradient = ctx.createLinearGradient(
				this.position.x,
				this.position.y,
				this.position.x,
				this.position.y + this.height
			);
			gradient.addColorStop(0, '#70a1ff');
			gradient.addColorStop(1, '#1e90ff');

			ctx.shadowBlur = 14;
			ctx.shadowColor = 'rgba(30, 144, 255, 0.4)';
			ctx.fillStyle = gradient;
			ctx.beginPath();
			ctx.roundRect(this.position.x, this.position.y, this.width, this.height, 8);
			ctx.fill();
			ctx.shadowBlur = 0;

			ctx.fillStyle = '#ffffff';
			ctx.fillRect(this.position.x + 6, this.position.y + 2, this.width - 12, 2.5);
		}
	}

	// ─── ĐỆM LÒ XO PHÓNG CAO (SPRING PAD) ───
	class SpringPad {
		position: Position;
		width = 40;
		height = 12;
		compressed = 0;

		constructor(x: number, y: number) {
			this.position = { x, y };
		}

		draw() {
			if (!ctx) return;

			const sy = this.position.y + (this.compressed > 0 ? 5 : 0);
			const h = this.height - (this.compressed > 0 ? 5 : 0);

			ctx.shadowBlur = 12;
			ctx.shadowColor = '#2ed573';
			ctx.fillStyle = '#2ed573';
			ctx.beginPath();
			ctx.roundRect(this.position.x, sy, this.width, h, 4);
			ctx.fill();
			ctx.shadowBlur = 0;

			ctx.fillStyle = '#ffa502';
			ctx.fillRect(this.position.x + 6, sy + 2, this.width - 12, 3);

			if (this.compressed > 0) this.compressed--;
		}
	}

	// ─── ĐỒNG XU VÀNG 3D ───
	class Coin {
		position: Position;
		radius = 11;
		collected = false;

		constructor(x: number, y: number) {
			this.position = { x, y };
		}

		draw(time: number) {
			if (this.collected || !ctx) return;

			const wobble = Math.sin(time * 0.005 + this.position.x) * 3;
			const scaleX = Math.abs(Math.cos(time * 0.004 + this.position.x));
			const sy = this.position.y + wobble;

			ctx.shadowBlur = 12;
			ctx.shadowColor = 'rgba(255, 211, 42, 0.7)';
			ctx.fillStyle = '#ffd32a';
			ctx.beginPath();
			ctx.ellipse(
				this.position.x,
				sy,
				Math.max(2.5, this.radius * scaleX),
				this.radius,
				0,
				0,
				Math.PI * 2
			);
			ctx.fill();
			ctx.shadowBlur = 0;

			ctx.fillStyle = '#ffffff';
			ctx.fillRect(this.position.x - 1, sy - 3.5, 2, 7);
		}
	}

	// ─── QUÁI VẬT TUẦN TRA (SLIME) ───
	class SlimeEnemy {
		position: Position;
		width = 34;
		height = 24;
		minX: number;
		maxX: number;
		speed = 1.6;
		alive = true;

		constructor(x: number, y: number, minX: number, maxX: number) {
			this.position = { x, y };
			this.minX = minX;
			this.maxX = maxX;
		}

		update() {
			if (!this.alive) return;
			this.position.x += this.speed;
			if (this.position.x > this.maxX) {
				this.position.x = this.maxX;
				this.speed = -Math.abs(this.speed);
			} else if (this.position.x < this.minX) {
				this.position.x = this.minX;
				this.speed = Math.abs(this.speed);
			}
		}

		draw(time: number) {
			if (!this.alive || !ctx) return;

			const wobble = Math.sin(time * 0.008 + this.position.x) * 3;
			ctx.shadowBlur = 10;
			ctx.shadowColor = 'rgba(255, 71, 87, 0.5)';
			ctx.fillStyle = '#ff4757';
			ctx.beginPath();
			ctx.ellipse(
				this.position.x + 17,
				this.position.y + 12,
				17 + wobble * 0.4,
				12 - wobble * 0.4,
				0,
				0,
				Math.PI * 2
			);
			ctx.fill();
			ctx.shadowBlur = 0;

			const eyeDir = this.speed > 0 ? 3 : -3;
			ctx.fillStyle = '#ffffff';
			ctx.fillRect(this.position.x + 10 + eyeDir, this.position.y + 6, 4, 6);
			ctx.fillRect(this.position.x + 20 + eyeDir, this.position.y + 6, 4, 6);
			ctx.fillStyle = '#1e293b';
			ctx.fillRect(this.position.x + 11 + eyeDir * 1.3, this.position.y + 8, 2, 3);
			ctx.fillRect(this.position.x + 21 + eyeDir * 1.3, this.position.y + 8, 2, 3);
		}
	}

	// ─── CỜ CHECKPOINT ───
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

	function triggerJump() {
		if (player.jumpCount < 2) {
			if (player.jumpCount === 0) {
				player.velocity.y = -jumpForce;
				soundJump();
				spawnDust(player.position.x + 20, player.position.y + player.height, '#ffffff', 6);
			} else {
				// Double Jump
				player.velocity.y = -jumpForce * 0.95;
				soundDoubleJump();
				spawnSparkles(player.position.x + 20, player.position.y + 20, '#4ECDC4', 8);
			}
			player.jumpCount++;
		}
	}

	function showCheckpoint(message: string) {
		checkpointMessage = message;
		checkpointVisible = true;

		if (hideCheckpointTimer) {
			clearTimeout(hideCheckpointTimer);
		}

		hideCheckpointTimer = setTimeout(() => {
			checkpointVisible = false;
		}, 2000);
	}

	function initGame() {
		platforms = LEVEL_PLATFORMS.map((platform) => new Platform(platform));
		checkpoints = LEVEL_CHECKPOINTS.map((checkpoint) => new Checkpoint(checkpoint));

		movingPlatforms = [
			new MovingPlatform(1400, 390, 160, 20, 180, 2),
			new MovingPlatform(2020, 260, 140, 20, 140, 1.8)
		];

		springPads = [new SpringPad(500, 508), new SpringPad(1360, 458)];

		coins = [
			new Coin(220, 560),
			new Coin(280, 560),
			new Coin(340, 560),
			new Coin(500, 350),
			new Coin(500, 290),
			new Coin(790, 380),
			new Coin(850, 380),
			new Coin(1100, 310),
			new Coin(1160, 310),
			new Coin(1360, 300),
			new Coin(1360, 240),
			new Coin(1480, 330),
			new Coin(1720, 330),
			new Coin(1780, 330),
			new Coin(2040, 260),
			new Coin(2100, 260),
			new Coin(2320, 370),
			new Coin(2380, 370)
		];
		totalCoins = coins.length;
		coinsCollected = 0;
		score = 0;
		lives = 3;

		enemies = [
			new SlimeEnemy(460, 496, 430, 590),
			new SlimeEnemy(1720, 371, 1680, 1840)
		];

		particles = [];
		scrollOffset = 0;
		checkpointVisible = false;
		checkpointMessage = '';
		gameWon = false;
		lastRespawn = { x: playerStart.x, y: playerStart.y, scrollOffset: 0 };
		player.reset();
	}

	function respawnPlayer() {
		lives--;
		if (lives <= 0) {
			showCheckpoint('Hết mạng! Khởi động lại màn chơi...');
			setTimeout(() => {
				initGame();
			}, 600);
			return;
		}

		showCheckpoint(`Rơi xuống vực! Còn lại ${lives} mạng.`);
		const diff = scrollOffset - lastRespawn.scrollOffset;
		platforms.forEach((p) => (p.position.x += diff));
		checkpoints.forEach((c) => (c.position.x += diff));
		movingPlatforms.forEach((m) => {
			m.position.x += diff;
			m.minX += diff;
			m.maxX += diff;
		});
		springPads.forEach((s) => (s.position.x += diff));
		coins.forEach((c) => (c.position.x += diff));
		enemies.forEach((e) => {
			e.position.x += diff;
			e.minX += diff;
			e.maxX += diff;
		});

		scrollOffset = lastRespawn.scrollOffset;
		player.position.x = lastRespawn.x;
		player.position.y = lastRespawn.y;
		player.velocity.x = 0;
		player.velocity.y = 0;
		player.jumpCount = 0;
	}

	function resizeCanvas() {
		if (!canvas) return;
		canvas.width = window.innerWidth;
		canvas.height = window.innerHeight;
		// Căn chỉnh camera khi màn hình có chiều cao thấp
		cameraY = canvas.height < 700 ? Math.max(0, 680 - canvas.height) : 0;
	}

	function drawBackground() {
		if (!ctx || !canvas) return;

		const sky = ctx.createLinearGradient(0, 0, 0, canvas.height);
		sky.addColorStop(0, '#2c315d');
		sky.addColorStop(1, '#171b38');
		ctx.fillStyle = sky;
		ctx.fillRect(0, 0, canvas.width, canvas.height);

		for (let i = 0; i < 35; i += 1) {
			const x =
				(((i * 147 - scrollOffset * 0.25) % (canvas.width + 160)) + canvas.width + 160) %
				(canvas.width + 160);
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

		let delta = 0;
		if (keys.right) {
			if (player.position.x < rightAnchor || scrollOffset >= maxScroll) {
				player.velocity.x = moveSpeed;
			} else {
				player.velocity.x = 0;
				delta = moveSpeed;
			}
		} else if (keys.left) {
			if (player.position.x > leftAnchor || scrollOffset <= 0) {
				player.velocity.x = -moveSpeed;
			} else {
				player.velocity.x = 0;
				delta = -moveSpeed;
			}
		} else {
			player.velocity.x = 0;
		}

		if (delta !== 0) {
			scrollOffset = Math.max(0, Math.min(scrollOffset + delta, maxScroll));
			platforms.forEach((p) => (p.position.x -= delta));
			checkpoints.forEach((c) => (c.position.x -= delta));
			movingPlatforms.forEach((m) => {
				m.position.x -= delta;
				m.minX -= delta;
				m.maxX -= delta;
			});
			springPads.forEach((s) => (s.position.x -= delta));
			coins.forEach((c) => (c.position.x -= delta));
			enemies.forEach((e) => {
				e.position.x -= delta;
				e.minX -= delta;
				e.maxX -= delta;
			});
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
				player.jumpCount = 0;
			}
		}

		for (const moving of movingPlatforms) {
			const isLanding =
				player.velocity.y >= 0 &&
				previousY + player.height <= moving.position.y &&
				player.position.y + player.height >= moving.position.y &&
				player.position.x + player.width >= moving.position.x &&
				player.position.x <= moving.position.x + moving.width;

			if (isLanding) {
				player.position.y = moving.position.y - player.height;
				player.velocity.y = 0;
				player.jumpCount = 0;
				player.position.x += moving.speed;
			}
		}

		for (const spring of springPads) {
			const isHitting =
				player.velocity.y >= 0 &&
				previousY + player.height <= spring.position.y &&
				player.position.y + player.height >= spring.position.y &&
				player.position.x + player.width >= spring.position.x &&
				player.position.x <= spring.position.x + spring.width;

			if (isHitting) {
				player.velocity.y = -21;
				player.jumpCount = 1;
				spring.compressed = 6;
				soundSpring();
				spawnSparkles(spring.position.x + 20, spring.position.y, '#2ed573', 10);
			}
		}
	}

	function checkCoinCollisions() {
		for (const coin of coins) {
			if (coin.collected) continue;
			const dx = player.position.x + 20 - coin.position.x;
			const dy = player.position.y + 20 - coin.position.y;
			if (dx * dx + dy * dy < 28 * 28) {
				coin.collected = true;
				coinsCollected++;
				score += 100;
				soundCoin();
				spawnSparkles(coin.position.x, coin.position.y, '#ffd32a', 8);
			}
		}
	}

	function checkEnemyCollisions(previousY: number) {
		for (const enemy of enemies) {
			if (!enemy.alive) continue;

			const isOverlapping =
				player.position.x + player.width >= enemy.position.x &&
				player.position.x <= enemy.position.x + enemy.width &&
				player.position.y + player.height >= enemy.position.y &&
				player.position.y <= enemy.position.y + enemy.height;

			if (isOverlapping) {
				if (player.velocity.y >= 0 && previousY + player.height <= enemy.position.y + 14) {
					enemy.alive = false;
					player.velocity.y = -10;
					player.jumpCount = 1;
					score += 250;
					soundStomp();
					spawnSparkles(enemy.position.x + 16, enemy.position.y + 12, '#ff4757', 12);
				} else {
					respawnPlayer();
					return;
				}
			}
		}
	}

	function checkCheckpointCollisions() {
		for (let i = 0; i < checkpoints.length; i++) {
			const checkpoint = checkpoints[i];
			const isTouching =
				player.position.x + player.width >= checkpoint.position.x &&
				player.position.x <= checkpoint.position.x + checkpoint.width &&
				player.position.y + player.height >= checkpoint.position.y &&
				player.position.y <= checkpoint.position.y + checkpoint.height;

			if (isTouching && !checkpoint.reached) {
				checkpoint.reached = true;
				soundCheckpoint();
				showCheckpoint(checkpoint.message);
				spawnSparkles(checkpoint.position.x + 14, checkpoint.position.y + 15, '#81f7c7', 14);

				lastRespawn = {
					x: player.position.x,
					y: player.position.y,
					scrollOffset
				};

				if (i === checkpoints.length - 1) {
					gameWon = true;
					soundVictory();
					spawnSparkles(checkpoint.position.x + 14, checkpoint.position.y, '#ffd32a', 30);
				}
			}
		}
	}

	function drawScene(time: number) {
		if (!ctx) return;

		ctx.save();
		if (cameraY > 0) {
			ctx.translate(0, -cameraY);
		}

		platforms.forEach((platform) => platform.draw());
		movingPlatforms.forEach((moving) => moving.draw());
		springPads.forEach((spring) => spring.draw());
		coins.forEach((coin) => coin.draw(time));
		enemies.forEach((enemy) => enemy.draw(time));
		checkpoints.forEach((checkpoint) => checkpoint.draw());
		drawParticles();
		player.draw();

		ctx.restore();
	}

	function animate(ts = 0) {
		if (!gameStarted || !canvas || !ctx) return;

		animationFrame = requestAnimationFrame(animate);
		drawBackground();
		handleWorldScroll();

		movingPlatforms.forEach((m) => m.update());
		enemies.forEach((e) => e.update());
		updateParticles();

		const previousY = player.position.y;
		const worldFloor = canvas.height + cameraY + 160;

		player.update(worldFloor);
		resolvePlatformCollisions(previousY);
		checkCoinCollisions();
		checkEnemyCollisions(previousY);
		checkCheckpointCollisions();

		if (player.position.y > canvas.height + cameraY + 100) {
			respawnPlayer();
			return;
		}

		drawScene(ts);
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

		if (event.key === ' ' || event.key === 'ArrowUp' || event.key.toLowerCase() === 'w') {
			event.preventDefault();
			triggerJump();
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
		drawScene(0);

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

<svelte:head>
	<title>Neon Platformer | Phiêu Lưu Cyberpunk 2D</title>
	<meta
		name="description"
		content="Trải nghiệm game nhảy bệ Neon Platformer cực cuốn với nhảy kép, đệm lò xo, tiền vàng và quái vật tuần tra."
	/>
</svelte:head>

<div class="platformer-wrapper">
	<!-- 
		THANH HEADER TRÊN CÙNG: 
		- Nút BackHome neo chắc chắn bên trái, KHÔNG BAO GIỜ trôi nổi che giao diện
		- Tích hợp trực tiếp bảng chỉ số điểm, xu, mạng sống
	-->
	<header class="top-nav">
		<div class="nav-left">
			<BackHome label="Trang chủ" />
		</div>

		<div class="nav-center">
			<span class="pulse-dot"></span>
			<span class="game-badge">NEON PLATFORMER</span>
		</div>

		<div class="nav-right">
			{#if gameStarted && !gameWon}
				<div class="hud-pill">
					<span class="hud-stat coins" title="Đồng xu vàng">🪙 {coinsCollected}/{totalCoins}</span>
					<span class="hud-sep">•</span>
					<span class="hud-stat score" title="Điểm số">⭐ {score}</span>
					<span class="hud-sep">•</span>
					<span class="hud-stat lives" title="Mạng sống">{'❤️'.repeat(Math.max(0, lives))}</span>
				</div>
			{/if}
		</div>
	</header>

	<!-- 
		CARD TRƯỚC KHI VÀO CHƠI ĐƯỢC THIẾT KẾ LẠI BẮT MẮT & HẤP DẪN:
		- Nhân vật 3D avatar lắc lư sinh động
		- Lưới tính năng nổi bật (Nhảy kép, 18 xu vàng, Quái vật Slime)
		- Bảng hướng dẫn phím bấm công thái học
		- Nút CTA phát sáng neon cuốn hút
	-->
	{#if !gameStarted}
		<div class="intro-card-overlay">
			<div class="intro-card">
				<!-- Hero Avatar nhún nhảy sinh động -->
				<div class="hero-character-box">
					<div class="hero-halo"></div>
					<div class="hero-cube">
						<div class="cube-eye left"></div>
						<div class="cube-eye right"></div>
					</div>
				</div>

				<div class="title-wrap">
					<span class="sub-badge">CYBERPUNK 2D ADVENTURE</span>
					<h1 class="main-title">NEON PLATFORMER</h1>
					<p class="author-credit">Thiết kế bởi <b>Nguyen Quy Coder</b></p>
				</div>

				<!-- Lưới 3 điểm nhấn gameplay đặc sắc -->
				<div class="feature-grid">
					<div class="feature-card">
						<span class="feat-icon">🚀</span>
						<div class="feat-text">
							<span class="feat-name">Nhảy Kép</span>
							<span class="feat-desc">Nhấn Space 2 lần liên tiếp</span>
						</div>
					</div>

					<div class="feature-card">
						<span class="feat-icon">🪙</span>
						<div class="feat-text">
							<span class="feat-name">18 Đồng Xu</span>
							<span class="feat-desc">Tích lũy điểm & đệm lò xo</span>
						</div>
					</div>

					<div class="feature-card">
						<span class="feat-icon">👾</span>
						<div class="feat-text">
							<span class="feat-name">Giẫm Quái</span>
							<span class="feat-desc">Nhảy đè Slime từ trên cao</span>
						</div>
					</div>
				</div>

				<!-- Hướng dẫn phím bấm trực quan -->
				<div class="controls-pill-grid">
					<div class="ctrl-row">
						<span class="ctrl-title">Di chuyển:</span>
						<div class="key-caps">
							<kbd>A</kbd><kbd>D</kbd> <span class="or">hoặc</span> <kbd>◀</kbd><kbd>▶</kbd>
						</div>
					</div>

					<div class="ctrl-row">
						<span class="ctrl-title">Nhảy / Nhảy kép:</span>
						<div class="key-caps">
							<kbd class="space-kbd">SPACE</kbd> <kbd>W</kbd> <kbd>▲</kbd>
						</div>
					</div>
				</div>

				<!-- Nút Bắt Đầu Rực Rỡ -->
				<div class="btn-wrap">
					<button type="button" class="btn-play-hero" onclick={startGame}>
						<span class="btn-shimmer"></span>
						<span class="btn-label">🎮 BẮT ĐẦU PHIÊU LƯU</span>
					</button>
				</div>
			</div>
		</div>
	{/if}

	<!-- MÀN HÌNH CHIẾN THẮNG KHI VỀ ĐÍCH -->
	{#if gameWon}
		<div class="intro-card-overlay">
			<div class="intro-card victory-mode">
				<div class="trophy-badge">🏆</div>
				<h1 class="main-title victory-title">CHIẾN THẮNG!</h1>
				<p class="victory-desc">Bạn đã xuất sắc chinh phục toàn bộ các bệ nhảy và về đích an toàn!</p>

				<div class="stats-summary">
					<div class="stat-box">
						<span class="stat-num">{coinsCollected} / {totalCoins}</span>
						<span class="stat-label">🪙 Đồng xu</span>
					</div>
					<div class="stat-box">
						<span class="stat-num">{score}</span>
						<span class="stat-label">⭐ Tổng điểm</span>
					</div>
					<div class="stat-box">
						<span class="stat-num">{'❤️'.repeat(lives)}</span>
						<span class="stat-label">Mạng còn lại</span>
					</div>
				</div>

				<div class="btn-wrap">
					<button type="button" class="btn-play-hero" onclick={startGame}>
						<span class="btn-label">↺ CHƠI LẠI MÀN NÀY</span>
					</button>
				</div>
			</div>
		</div>
	{/if}

	<!-- BẢNG THÔNG BÁO CHECKPOINT -->
	{#if checkpointVisible}
		<div class="checkpoint-screen">
			<span class="cp-icon">🚩</span>
			<div class="cp-text">
				<span class="cp-title">CHECKPOINT</span>
				<p class="cp-desc">{checkpointMessage}</p>
			</div>
		</div>
	{/if}

	<!-- Nút cảm ứng trên Mobile (Tự động ẩn trên máy tính) -->
	{#if gameStarted && !gameWon}
		<div class="touch-controls" aria-label="Nút điều khiển cảm ứng">
			<div class="touch-dpad">
				<button
					type="button"
					class="touch-btn"
					onpointerdown={() => (keys.left = true)}
					onpointerup={() => (keys.left = false)}
					onpointerleave={() => (keys.left = false)}
					aria-label="Sang trái"
				>
					◀
				</button>
				<button
					type="button"
					class="touch-btn"
					onpointerdown={() => (keys.right = true)}
					onpointerup={() => (keys.right = false)}
					onpointerleave={() => (keys.right = false)}
					aria-label="Sang phải"
				>
					▶
				</button>
			</div>

			<div class="touch-actions">
				<button
					type="button"
					class="touch-btn touch-jump"
					onpointerdown={(e) => {
						e.preventDefault();
						triggerJump();
					}}
					aria-label="Nhảy"
				>
					▲
				</button>
			</div>
		</div>
	{/if}

	<canvas bind:this={canvas} class:visible={gameStarted}></canvas>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@500;600;700;800;900&family=JetBrains+Mono:wght@600;700;800&display=swap');

	.platformer-wrapper {
		position: fixed;
		inset: 0;
		width: 100vw;
		height: 100vh;
		background: #171b38;
		overflow: hidden;
		user-select: none;
		-webkit-user-select: none;
		font-family: 'Plus Jakarta Sans', sans-serif;
		color: #ffffff;
	}

	/* =================================================
	   TOP NAV (Khắc phục 100% việc nút BackHome che giao diện)
	   ================================================= */
	.top-nav {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		height: 56px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 1.25rem;
		background: rgba(15, 20, 45, 0.75);
		border-bottom: 1px solid rgba(255, 255, 255, 0.08);
		backdrop-filter: blur(14px);
		-webkit-backdrop-filter: blur(14px);
		z-index: 50;
		box-sizing: border-box;
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
		padding: 0.42rem 0.95rem !important;
		font-size: 0.82rem !important;
		font-weight: 700 !important;
		border-radius: 999px !important;
		background: rgba(255, 255, 255, 0.06) !important;
		border: 1px solid rgba(255, 255, 255, 0.12) !important;
		color: #ffffff !important;
		transition: all 0.2s ease !important;
	}

	:global(.top-nav .back-home:hover) {
		background: rgba(78, 205, 196, 0.2) !important;
		border-color: rgba(78, 205, 196, 0.5) !important;
		transform: translateY(-1px) !important;
	}

	.nav-center {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	.pulse-dot {
		width: 7px;
		height: 7px;
		border-radius: 50%;
		background-color: #4ecdc4;
		box-shadow: 0 0 10px #4ecdc4;
		animation: pulseDot 2s infinite;
	}

	.game-badge {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.78rem;
		font-weight: 800;
		letter-spacing: 0.16em;
		color: #cbd5e1;
	}

	.nav-right {
		display: flex;
		align-items: center;
	}

	.hud-pill {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.35rem 0.95rem;
		border-radius: 999px;
		background: rgba(18, 24, 52, 0.85);
		border: 1px solid rgba(78, 205, 196, 0.3);
		box-shadow: 0 0 16px rgba(78, 205, 196, 0.15);
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.78rem;
		font-weight: 700;
	}

	.hud-stat.coins { color: #ffd32a; }
	.hud-stat.score { color: #4ecdc4; }
	.hud-sep { color: rgba(255, 255, 255, 0.2); }

	/* =================================================
	   CARD TRƯỚC KHI CHƠI (REDESIGNED BẮT MẮT & CUỐN HÚT)
	   ================================================= */
	.intro-card-overlay {
		position: absolute;
		inset: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 4.5rem 1rem 1.5rem;
		box-sizing: border-box;
		z-index: 40;
		overflow-y: auto;
	}

	.intro-card {
		position: relative;
		width: 100%;
		max-width: 520px;
		background: rgba(19, 24, 56, 0.82);
		border: 1px solid rgba(78, 205, 196, 0.3);
		border-radius: 24px;
		padding: 2.25rem 2rem 1.75rem;
		text-align: center;
		backdrop-filter: blur(24px);
		-webkit-backdrop-filter: blur(24px);
		box-shadow: 
			0 25px 60px rgba(0, 0, 0, 0.55),
			0 0 40px rgba(78, 205, 196, 0.12),
			inset 0 1px 0 rgba(255, 255, 255, 0.15);
		animation: cardSlideUp 0.4s cubic-bezier(0.16, 1, 0.3, 1);
		box-sizing: border-box;
	}

	/* Avatar Nhân Vật Nhún Nhảy */
	.hero-character-box {
		position: relative;
		width: 54px;
		height: 54px;
		margin: 0 auto 1rem;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.hero-halo {
		position: absolute;
		inset: -8px;
		border-radius: 16px;
		background: radial-gradient(circle, rgba(78, 205, 196, 0.5) 0%, transparent 70%);
		filter: blur(6px);
		animation: haloPulse 2.5s infinite alternate;
	}

	.hero-cube {
		position: relative;
		width: 44px;
		height: 44px;
		border-radius: 10px;
		background: #4ecdc4;
		box-shadow: 0 0 20px rgba(78, 205, 196, 0.6);
		animation: heroBob 2.2s ease-in-out infinite alternate;
	}

	.cube-eye {
		position: absolute;
		top: 12px;
		width: 8px;
		height: 9px;
		background: #ffffff;
		border-radius: 2px;
	}
	.cube-eye.left { left: 24px; }
	.cube-eye.right { left: 34px; }

	.cube-eye::after {
		content: '';
		position: absolute;
		top: 2px;
		left: 2px;
		width: 4px;
		height: 5px;
		background: #1e293b;
		border-radius: 1px;
	}

	/* Tiêu đề & Credit */
	.title-wrap {
		margin-bottom: 1.5rem;
	}

	.sub-badge {
		display: inline-block;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.65rem;
		font-weight: 800;
		letter-spacing: 0.22em;
		color: #4ecdc4;
		background: rgba(78, 205, 196, 0.12);
		border: 1px solid rgba(78, 205, 196, 0.25);
		padding: 0.25rem 0.65rem;
		border-radius: 999px;
		margin-bottom: 0.5rem;
	}

	.main-title {
		margin: 0;
		font-size: clamp(1.8rem, 5vw, 2.4rem);
		font-weight: 900;
		letter-spacing: -0.02em;
		background: linear-gradient(135deg, #ffffff 0%, #4ecdc4 55%, #ffe66d 100%);
		-webkit-background-clip: text;
		background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	.author-credit {
		margin: 0.35rem 0 0;
		font-size: 0.8rem;
		color: #94a3b8;
	}

	.author-credit b {
		color: #f1f5f9;
	}

	/* Lưới 3 điểm nhấn Feature */
	.feature-grid {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 0.75rem;
		margin-bottom: 1.25rem;
	}

	.feature-card {
		background: rgba(255, 255, 255, 0.04);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 12px;
		padding: 0.75rem 0.5rem;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.35rem;
		transition: all 0.2s ease;
	}

	.feature-card:hover {
		background: rgba(78, 205, 196, 0.08);
		border-color: rgba(78, 205, 196, 0.3);
		transform: translateY(-2px);
	}

	.feat-icon {
		font-size: 1.4rem;
	}

	.feat-name {
		display: block;
		font-size: 0.75rem;
		font-weight: 700;
		color: #f8fafc;
	}

	.feat-desc {
		display: block;
		font-size: 0.62rem;
		color: #94a3b8;
		line-height: 1.3;
	}

	/* Bảng Hướng Dẫn Phím Bấm */
	.controls-pill-grid {
		background: rgba(10, 14, 34, 0.55);
		border: 1px solid rgba(255, 255, 255, 0.06);
		border-radius: 14px;
		padding: 0.75rem 1rem;
		margin-bottom: 1.5rem;
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.ctrl-row {
		display: flex;
		align-items: center;
		justify-content: space-between;
		font-size: 0.78rem;
	}

	.ctrl-title {
		color: #94a3b8;
		font-weight: 600;
	}

	.key-caps {
		display: flex;
		align-items: center;
		gap: 0.35rem;
	}

	.key-caps kbd {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		background: #232a52;
		border: 1px solid rgba(255, 255, 255, 0.16);
		border-radius: 6px;
		padding: 0.15rem 0.45rem;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.72rem;
		font-weight: 700;
		color: #ffe66d;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
	}

	.key-caps .space-kbd {
		padding: 0.15rem 0.65rem;
		color: #4ecdc4;
	}

	.key-caps .or {
		font-size: 0.68rem;
		color: #64748b;
	}

	/* Nút Bắt Đầu Rực Rỡ */
	.btn-wrap {
		display: flex;
		justify-content: center;
	}

	.btn-play-hero {
		position: relative;
		width: 100%;
		padding: 0.95rem 2rem;
		border: none;
		border-radius: 999px;
		background: linear-gradient(135deg, #ee5253 0%, #ff7675 60%, #ffa502 100%);
		color: #ffffff;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.95rem;
		font-weight: 800;
		letter-spacing: 0.08em;
		cursor: pointer;
		overflow: hidden;
		box-shadow: 
			0 10px 30px rgba(238, 82, 83, 0.4),
			0 0 20px rgba(255, 118, 117, 0.3);
		transition: all 0.25s cubic-bezier(0.175, 0.885, 0.32, 1.275);
	}

	.btn-play-hero:hover {
		transform: translateY(-2px) scale(1.02);
		box-shadow: 
			0 15px 35px rgba(238, 82, 83, 0.55),
			0 0 30px rgba(255, 118, 117, 0.45);
	}

	.btn-play-hero:active {
		transform: scale(0.97);
	}

	.btn-shimmer {
		position: absolute;
		top: 0;
		left: -100%;
		width: 60%;
		height: 100%;
		background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
		transform: skewX(-20deg);
		animation: shimmerLoop 3s infinite;
	}

	/* =================================================
	   MÀN HÌNH CHIẾN THẮNG (VICTORY SCREEN)
	   ================================================= */
	.victory-mode {
		border-color: rgba(255, 230, 109, 0.4);
		box-shadow: 0 25px 60px rgba(0, 0, 0, 0.6), 0 0 50px rgba(255, 230, 109, 0.2);
	}

	.trophy-badge {
		font-size: 3rem;
		margin-bottom: 0.5rem;
		animation: trophyBounce 1.5s ease-in-out infinite alternate;
	}

	.victory-title {
		background: linear-gradient(135deg, #ffe66d 0%, #ffd32a 50%, #ff9f1c 100%);
		-webkit-background-clip: text;
		background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	.victory-desc {
		font-size: 0.85rem;
		color: #cbd5e1;
		margin: 0.5rem 0 1.25rem;
	}

	.stats-summary {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 0.75rem;
		margin-bottom: 1.5rem;
	}

	.stat-box {
		background: rgba(255, 255, 255, 0.05);
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 12px;
		padding: 0.75rem 0.5rem;
	}

	.stat-num {
		display: block;
		font-family: 'JetBrains Mono', monospace;
		font-size: 1.15rem;
		font-weight: 800;
		color: #ffe66d;
	}

	.stat-label {
		display: block;
		font-size: 0.68rem;
		color: #94a3b8;
		margin-top: 0.25rem;
	}

	/* =================================================
	   TOAST CHECKPOINT
	   ================================================= */
	.checkpoint-screen {
		position: fixed;
		top: 70px;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		align-items: center;
		gap: 0.75rem;
		padding: 0.65rem 1.25rem;
		border-radius: 12px;
		background: rgba(18, 25, 56, 0.9);
		border: 1px solid rgba(78, 205, 196, 0.5);
		backdrop-filter: blur(14px);
		box-shadow: 0 10px 30px rgba(0, 0, 0, 0.4), 0 0 20px rgba(78, 205, 196, 0.25);
		z-index: 55;
		animation: slideDownToast 0.35s cubic-bezier(0.16, 1, 0.3, 1);
	}

	.cp-icon {
		font-size: 1.25rem;
	}

	.cp-title {
		display: block;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.65rem;
		font-weight: 800;
		letter-spacing: 0.15em;
		color: #4ecdc4;
	}

	.cp-desc {
		margin: 0;
		font-size: 0.8rem;
		font-weight: 600;
		color: #f1f5f9;
	}

	/* =================================================
	   MOBILE TOUCH CONTROLS
	   ================================================= */
	.touch-controls {
		position: fixed;
		bottom: 24px;
		left: 20px;
		right: 20px;
		display: flex;
		justify-content: space-between;
		align-items: flex-end;
		z-index: 30;
		pointer-events: none;
	}

	.touch-dpad,
	.touch-actions {
		display: flex;
		gap: 12px;
		pointer-events: auto;
	}

	.touch-btn {
		width: 58px;
		height: 58px;
		border-radius: 16px;
		background: rgba(25, 32, 68, 0.75);
		border: 1px solid rgba(255, 255, 255, 0.18);
		color: #ffffff;
		font-size: 1.5rem;
		display: flex;
		align-items: center;
		justify-content: center;
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
		touch-action: manipulation;
		cursor: pointer;
		transition: transform 0.1s ease;
	}

	.touch-btn:active {
		transform: scale(0.9);
		background: rgba(78, 205, 196, 0.35);
		border-color: #4ecdc4;
	}

	.touch-jump {
		width: 68px;
		height: 68px;
		border-radius: 50%;
		background: rgba(238, 82, 83, 0.8);
		border-color: rgba(238, 82, 83, 0.95);
		box-shadow: 0 4px 20px rgba(238, 82, 83, 0.45);
	}

	.touch-jump:active {
		background: #ee5253;
	}

	@media (hover: hover) and (pointer: fine) {
		.touch-controls {
			display: none;
		}
	}

	/* Canvas */
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

	/* Keyframes */
	@keyframes cardSlideUp {
		from {
			opacity: 0;
			transform: translateY(20px) scale(0.97);
		}
		to {
			opacity: 1;
			transform: translateY(0) scale(1);
		}
	}

	@keyframes heroBob {
		from { transform: translateY(0); }
		to { transform: translateY(-8px); }
	}

	@keyframes haloPulse {
		from { transform: scale(0.9); opacity: 0.6; }
		to { transform: scale(1.15); opacity: 0.9; }
	}

	@keyframes shimmerLoop {
		0% { left: -100%; }
		40%, 100% { left: 140%; }
	}

	@keyframes trophyBounce {
		from { transform: scale(1); }
		to { transform: scale(1.1) rotate(4deg); }
	}

	@keyframes slideDownToast {
		from {
			opacity: 0;
			transform: translate(-50%, -20px);
		}
		to {
			opacity: 1;
			transform: translate(-50%, 0);
		}
	}

	@keyframes pulseDot {
		0%, 100% { transform: scale(1); opacity: 1; }
		50% { transform: scale(1.3); opacity: 0.6; }
	}

	/* Responsive */
	@media (max-width: 640px) {
		.top-nav {
			padding: 0 0.85rem;
		}

		.game-badge {
			display: none;
		}

		.hud-pill {
			font-size: 0.72rem;
			padding: 0.3rem 0.75rem;
		}

		.intro-card {
			padding: 1.5rem 1.25rem 1.25rem;
		}

		.feature-grid {
			gap: 0.5rem;
		}

		.feat-desc {
			display: none;
		}

		.feat-name {
			font-size: 0.7rem;
		}
	}
</style>
