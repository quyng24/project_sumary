<svelte:options runes={true} />

<script lang="ts">
	import { onMount } from 'svelte';
	import { resolve } from '$app/paths';

	// ─── HẰNG SỐ VẬT LÝ VÀ TRÒ CHƠI ───
	const CANVAS_WIDTH = 800;
	const CANVAS_HEIGHT = 280;
	const GROUND_Y = 225;

	const GRAVITY = 0.65;
	const JUMP_FORCE = -12.5;
	const DUCK_DROP_FORCE = 8;
	const INITIAL_SPEED = 7.0;
	const MAX_SPEED = 14.5;
	const ACCELERATION = 0.0012;

	let canvas = $state<HTMLCanvasElement | null>(null);
	let ctx: CanvasRenderingContext2D | null = null;

	let gameStarted = $state(false);
	let gameOver = $state(false);
	let isPaused = $state(false);
	let soundOn = $state(true);

	let score = $state(0);
	let highScore = $state(0);
	let speed = $state(INITIAL_SPEED);
	let distanceTraveled = 0;

	// Chu kỳ Ngày / Đêm (Cứ mỗi 700 điểm chuyển sang ban đêm)
	let isNight = $state(false);
	let nightFactor = $state(0); // 0 (Ngày) -> 1 (Đêm)

	// Nhấp nháy điểm khi đạt mốc 100 điểm
	let blinkScoreTimer = 0;
	let lastMilestone = 0;

	// Trạng thái nhân vật T-Rex
	const dino = {
		x: 50,
		y: GROUND_Y - 47,
		width: 44,
		height: 47,
		vy: 0,
		isJumping: false,
		isDucking: false,
		legFrame: 0,
		legTimer: 0,
		reset() {
			this.x = 50;
			this.y = GROUND_Y - 47;
			this.width = 44;
			this.height = 47;
			this.vy = 0;
			this.isJumping = false;
			this.isDucking = false;
			this.legFrame = 0;
			this.legTimer = 0;
		}
	};

	// ─── CHƯỚNG NGẠI VẬT & MÔI TRƯỜNG ───
	interface Obstacle {
		x: number;
		y: number;
		width: number;
		height: number;
		type: 'cactus_small' | 'cactus_large' | 'cactus_group' | 'bird';
		birdFrame?: number;
		birdTimer?: number;
	}

	interface Cloud {
		x: number;
		y: number;
		speed: number;
	}

	interface Star {
		x: number;
		y: number;
		size: number;
		alpha: number;
	}

	interface GroundBump {
		x: number;
		type: number;
	}

	let obstacles: Obstacle[] = [];
	let clouds: Cloud[] = [];
	let stars: Star[] = [];
	let groundBumps: GroundBump[] = [];
	let nextObstacleDistance = 300;
	let animationFrameId = 0;
	let audioCtx: AudioContext | null = null;

	// ─── WEB AUDIO API SYNTHESIZER ───
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
		if (!soundOn) return;
		try {
			const c = getAudioCtx();
			if (!c) return;
			const osc = c.createOscillator();
			const gain = c.createGain();
			osc.connect(gain);
			gain.connect(c.destination);
			osc.type = 'square';
			osc.frequency.setValueAtTime(440, c.currentTime);
			osc.frequency.exponentialRampToValueAtTime(880, c.currentTime + 0.08);
			gain.gain.setValueAtTime(0.15, c.currentTime);
			gain.gain.exponentialRampToValueAtTime(0.001, c.currentTime + 0.08);
			osc.start();
			osc.stop(c.currentTime + 0.08);
		} catch (_) {}
	}

	function soundMilestone() {
		if (!soundOn) return;
		try {
			const c = getAudioCtx();
			if (!c) return;
			const t = c.currentTime;
			const osc = c.createOscillator();
			const gain = c.createGain();
			osc.connect(gain);
			gain.connect(c.destination);
			osc.type = 'square';
			osc.frequency.setValueAtTime(587.33, t); // D5
			osc.frequency.setValueAtTime(880, t + 0.1); // A5
			gain.gain.setValueAtTime(0.18, t);
			gain.gain.exponentialRampToValueAtTime(0.001, t + 0.28);
			osc.start(t);
			osc.stop(t + 0.28);
		} catch (_) {}
	}

	function soundDie() {
		if (!soundOn) return;
		try {
			const c = getAudioCtx();
			if (!c) return;
			const osc = c.createOscillator();
			const gain = c.createGain();
			osc.connect(gain);
			gain.connect(c.destination);
			osc.type = 'sawtooth';
			osc.frequency.setValueAtTime(220, c.currentTime);
			osc.frequency.exponentialRampToValueAtTime(80, c.currentTime + 0.2);
			gain.gain.setValueAtTime(0.2, c.currentTime);
			gain.gain.exponentialRampToValueAtTime(0.001, c.currentTime + 0.22);
			osc.start();
			osc.stop(c.currentTime + 0.22);
		} catch (_) {}
	}

	// ─── KHỞI TẠO GAME ───
	function initGame() {
		dino.reset();
		speed = INITIAL_SPEED;
		score = 0;
		distanceTraveled = 0;
		obstacles = [];
		clouds = [
			{ x: 120, y: 45, speed: 0.6 },
			{ x: 380, y: 70, speed: 0.8 },
			{ x: 620, y: 35, speed: 0.5 }
		];
		stars = [
			{ x: 100, y: 30, size: 2, alpha: 0.7 },
			{ x: 250, y: 60, size: 1.5, alpha: 0.5 },
			{ x: 400, y: 25, size: 2, alpha: 0.8 },
			{ x: 550, y: 55, size: 1.5, alpha: 0.6 },
			{ x: 700, y: 35, size: 2, alpha: 0.9 }
		];
		groundBumps = [];
		for (let i = 0; i < 40; i++) {
			groundBumps.push({ x: i * 25 + Math.random() * 10, type: Math.floor(Math.random() * 3) });
		}
		nextObstacleDistance = 350;
		gameOver = false;
		isPaused = false;
		blinkScoreTimer = 0;
		lastMilestone = 0;
		isNight = false;
		nightFactor = 0;
	}

	function jump() {
		if (gameOver) {
			initGame();
			gameStarted = true;
			return;
		}
		if (!gameStarted) {
			gameStarted = true;
			return;
		}
		if (!dino.isJumping) {
			dino.vy = JUMP_FORCE;
			dino.isJumping = true;
			soundJump();
		}
	}

	function duck(start: boolean) {
		if (gameOver || !gameStarted) return;
		dino.isDucking = start;
		if (start && dino.isJumping) {
			// Fast drop khi ấn xuống lúc đang trên không
			dino.vy += DUCK_DROP_FORCE;
		}
		if (start) {
			dino.width = 59;
			dino.height = 30;
			dino.y = GROUND_Y - 30;
		} else {
			dino.width = 44;
			dino.height = 47;
			if (!dino.isJumping) {
				dino.y = GROUND_Y - 47;
			}
		}
	}

	function handleKeyDown(e: KeyboardEvent) {
		if (e.code === 'Space' || e.code === 'ArrowUp' || e.code === 'KeyW') {
			e.preventDefault();
			jump();
		} else if (e.code === 'ArrowDown' || e.code === 'KeyS') {
			e.preventDefault();
			duck(true);
		} else if (e.code === 'KeyP') {
			e.preventDefault();
			if (gameStarted && !gameOver) isPaused = !isPaused;
		}
	}

	function handleKeyUp(e: KeyboardEvent) {
		if (e.code === 'ArrowDown' || e.code === 'KeyS') {
			duck(false);
		}
	}

	// ─── SINH CHƯỚNG NGẠI VẬT THÔNG MINH ───
	function spawnObstacle() {
		// Chim Pterodactyl chỉ xuất hiện khi điểm > 350
		const canSpawnBird = score > 350;
		const rand = Math.random();

		if (canSpawnBird && rand < 0.35) {
			// 3 mức độ cao của Chim: Cao (chạy thẳng), Trung (phải cúi), Thấp (phải nhảy)
			const birdYChoices = [GROUND_Y - 72, GROUND_Y - 48, GROUND_Y - 24];
			const birdY = birdYChoices[Math.floor(Math.random() * birdYChoices.length)];
			obstacles.push({
				x: CANVAS_WIDTH + 50,
				y: birdY,
				width: 44,
				height: 32,
				type: 'bird',
				birdFrame: 0,
				birdTimer: 0
			});
		} else {
			// Cây xương rồng (Xương rồng nhỏ, to, hoặc cụm đôi/ba)
			const cactusRand = Math.random();
			if (cactusRand < 0.45) {
				obstacles.push({
					x: CANVAS_WIDTH + 50,
					y: GROUND_Y - 36,
					width: 17,
					height: 36,
					type: 'cactus_small'
				});
			} else if (cactusRand < 0.75) {
				obstacles.push({
					x: CANVAS_WIDTH + 50,
					y: GROUND_Y - 48,
					width: 25,
					height: 48,
					type: 'cactus_large'
				});
			} else {
				obstacles.push({
					x: CANVAS_WIDTH + 50,
					y: GROUND_Y - 36,
					width: 38,
					height: 36,
					type: 'cactus_group'
				});
			}
		}

		// Khoảng cách giữa 2 chướng ngại vật tỉ lệ nghịch với tốc độ
		nextObstacleDistance = 240 + Math.random() * 200 + speed * 12;
	}

	// ─── KIỂM TRA VA CHẠM AABB ĐƯỢC BO VIỀN CHÍNH XÁC ───
	function checkCollision(obs: Obstacle): boolean {
		const padX = 6;
		const padY = 5;

		const dinoBox = {
			left: dino.x + padX,
			right: dino.x + dino.width - padX,
			top: dino.y + padY,
			bottom: dino.y + dino.height - padY
		};

		const obsBox = {
			left: obs.x + padX,
			right: obs.x + obs.width - padX,
			top: obs.y + padY,
			bottom: obs.y + obs.height - padY
		};

		return (
			dinoBox.right > obsBox.left &&
			dinoBox.left < obsBox.right &&
			dinoBox.bottom > obsBox.top &&
			dinoBox.top < obsBox.bottom
		);
	}

	// ─── HÀM VẼ SPRITE PIXEL NGUYÊN BẢN CHROME ───
	function getColor() {
		// Màu sắc tự đảo theo chế độ ngày đêm
		return isNight ? '#e2e8f0' : '#535353';
	}

	function drawDino(c: CanvasRenderingContext2D) {
		const color = getColor();
		c.fillStyle = color;

		const x = dino.x;
		const y = dino.y;

		if (gameOver) {
			// Khủng long va chạm (Mắt mở to bàng hoàng dạng chữ X)
			drawPixelDinoStanding(c, x, y, color);
			// Vẽ mắt chữ X
			c.fillStyle = isNight ? '#121218' : '#ffffff';
			c.fillRect(x + 24, y + 4, 8, 8);
			c.fillStyle = color;
			c.fillRect(x + 25, y + 5, 2, 2);
			c.fillRect(x + 29, y + 5, 2, 2);
			c.fillRect(x + 27, y + 7, 2, 2);
			c.fillRect(x + 25, y + 9, 2, 2);
			c.fillRect(x + 29, y + 9, 2, 2);
			return;
		}

		if (dino.isDucking) {
			// Tư thế cúi rạp người chạy
			drawPixelDinoDucking(c, x, y, color, dino.legFrame);
			return;
		}

		if (dino.isJumping) {
			// Tư thế nhảy (2 chân co lại)
			drawPixelDinoStanding(c, x, y, color);
			return;
		}

		// Tư thế chạy (Chân đổi luân phiên)
		drawPixelDinoRunning(c, x, y, color, dino.legFrame);
	}

	// Vẽ T-Rex tư thế đứng chuẩn pixel Chrome
	function drawPixelDinoStanding(c: CanvasRenderingContext2D, x: number, y: number, color: string) {
		c.fillStyle = color;
		// Đầu & Mõm
		c.fillRect(x + 22, y, 22, 18);
		// Miệng
		c.clearRect(x + 22, y + 14, 14, 4);
		// Mắt
		c.fillStyle = isNight ? '#121218' : '#ffffff';
		c.fillRect(x + 26, y + 4, 4, 4);

		c.fillStyle = color;
		// Cổ & Thân
		c.fillRect(x + 18, y + 18, 14, 18);
		c.fillRect(x + 10, y + 22, 18, 16);
		// Đuôi
		c.fillRect(x, y + 20, 10, 6);
		c.fillRect(x + 2, y + 26, 8, 4);
		// Tay nhỏ
		c.fillRect(x + 32, y + 24, 6, 4);
		c.fillRect(x + 36, y + 26, 2, 4);

		// Chân đứng
		c.fillRect(x + 16, y + 38, 4, 9);
		c.fillRect(x + 16, y + 45, 8, 2);
		c.fillRect(x + 26, y + 38, 4, 9);
		c.fillRect(x + 26, y + 45, 8, 2);
	}

	// Vẽ T-Rex chạy (Leg frame 0 / 1)
	function drawPixelDinoRunning(
		c: CanvasRenderingContext2D,
		x: number,
		y: number,
		color: string,
		frame: number
	) {
		c.fillStyle = color;
		// Đầu & Thân
		c.fillRect(x + 22, y, 22, 18);
		c.clearRect(x + 22, y + 14, 14, 4);

		c.fillStyle = isNight ? '#121218' : '#ffffff';
		c.fillRect(x + 26, y + 4, 4, 4);

		c.fillStyle = color;
		c.fillRect(x + 18, y + 18, 14, 18);
		c.fillRect(x + 10, y + 22, 18, 16);
		c.fillRect(x, y + 20, 10, 6);
		c.fillRect(x + 2, y + 26, 8, 4);
		c.fillRect(x + 32, y + 24, 6, 4);
		c.fillRect(x + 36, y + 26, 2, 4);

		// Chân bước luân phiên
		if (frame === 0) {
			c.fillRect(x + 16, y + 38, 4, 9);
			c.fillRect(x + 16, y + 45, 8, 2);
			c.fillRect(x + 26, y + 38, 6, 4);
			c.fillRect(x + 30, y + 42, 4, 4);
		} else {
			c.fillRect(x + 14, y + 38, 6, 4);
			c.fillRect(x + 18, y + 42, 4, 4);
			c.fillRect(x + 26, y + 38, 4, 9);
			c.fillRect(x + 26, y + 45, 8, 2);
		}
	}

	// Vẽ T-Rex cúi người (Ducking)
	function drawPixelDinoDucking(
		c: CanvasRenderingContext2D,
		x: number,
		y: number,
		color: string,
		frame: number
	) {
		c.fillStyle = color;
		// Thân dài nằm ngang
		c.fillRect(x + 6, y + 8, 38, 16);
		// Đuôi vểnh
		c.fillRect(x, y + 4, 8, 6);
		// Đầu nằm ngang
		c.fillRect(x + 38, y, 21, 14);
		c.clearRect(x + 38, y + 10, 14, 4);

		// Mắt
		c.fillStyle = isNight ? '#121218' : '#ffffff';
		c.fillRect(x + 44, y + 3, 4, 4);

		c.fillStyle = color;
		// Chân bò luân phiên
		if (frame === 0) {
			c.fillRect(x + 18, y + 22, 4, 8);
			c.fillRect(x + 18, y + 28, 8, 2);
			c.fillRect(x + 34, y + 22, 6, 4);
			c.fillRect(x + 38, y + 26, 4, 4);
		} else {
			c.fillRect(x + 16, y + 22, 6, 4);
			c.fillRect(x + 20, y + 26, 4, 4);
			c.fillRect(x + 34, y + 22, 4, 8);
			c.fillRect(x + 34, y + 28, 8, 2);
		}
	}

	// Vẽ Cây Xương Rồng Nhỏ
	function drawSmallCactus(c: CanvasRenderingContext2D, x: number, y: number, color: string) {
		c.fillStyle = color;
		// Thân chính
		c.fillRect(x + 6, y, 6, 36);
		// Nhánh trái
		c.fillRect(x, y + 8, 4, 16);
		c.fillRect(x, y + 20, 6, 4);
		// Nhánh phải
		c.fillRect(x + 13, y + 12, 4, 14);
		c.fillRect(x + 11, y + 22, 4, 4);
	}

	// Vẽ Cây Xương Rồng To
	function drawLargeCactus(c: CanvasRenderingContext2D, x: number, y: number, color: string) {
		c.fillStyle = color;
		// Thân to
		c.fillRect(x + 9, y, 8, 48);
		// Nhánh trái cao
		c.fillRect(x, y + 12, 5, 22);
		c.fillRect(x, y + 30, 9, 5);
		// Nhánh phải
		c.fillRect(x + 19, y + 16, 5, 20);
		c.fillRect(x + 16, y + 32, 6, 5);
	}

	// Vẽ Cụm Xương Rồng
	function drawCactusGroup(c: CanvasRenderingContext2D, x: number, y: number, color: string) {
		drawSmallCactus(c, x, y, color);
		drawSmallCactus(c, x + 20, y, color);
	}

	// Vẽ Chim Pterodactyl (Vỗ cánh 2 frame)
	function drawBird(
		c: CanvasRenderingContext2D,
		x: number,
		y: number,
		color: string,
		frame: number
	) {
		c.fillStyle = color;
		// Thân chim & Mỏ nhọn
		c.fillRect(x + 14, y + 12, 18, 8);
		c.fillRect(x + 32, y + 14, 12, 4);
		c.fillRect(x + 4, y + 10, 10, 6);

		// Cánh vỗ lên hoặc vỗ xuống
		if (frame === 0) {
			// Cánh chếch lên
			c.fillRect(x + 18, y, 6, 12);
			c.fillRect(x + 22, y - 6, 4, 8);
		} else {
			// Cánh chúc xuống
			c.fillRect(x + 18, y + 18, 6, 14);
			c.fillRect(x + 22, y + 28, 4, 6);
		}
	}

	// Vẽ Đám Mây Pixel
	function drawCloud(c: CanvasRenderingContext2D, x: number, y: number, color: string) {
		c.fillStyle = color;
		c.fillRect(x + 18, y, 28, 6);
		c.fillRect(x + 6, y + 6, 48, 6);
		c.fillRect(x, y + 12, 56, 6);
	}

	// Vẽ Mặt Trăng Khuyết (Ban đêm)
	function drawMoon(c: CanvasRenderingContext2D, x: number, y: number) {
		c.fillStyle = '#e2e8f0';
		c.beginPath();
		c.arc(x, y, 16, 0, Math.PI * 2);
		c.fill();
		// Che tạo hình trăng khuyết
		c.fillStyle = '#1e1e24';
		c.beginPath();
		c.arc(x + 6, y - 4, 14, 0, Math.PI * 2);
		c.fill();
	}

	// ─── GAME LOOP CHÍNH ───
	function update() {
		if (gameOver || isPaused || !gameStarted) return;

		// Tăng tốc dần theo khoảng cách
		speed = Math.min(MAX_SPEED, speed + ACCELERATION);
		distanceTraveled += speed;
		score = Math.floor(distanceTraveled * 0.1);

		// Chuyển chế độ ngày/đêm mỗi 700 điểm
		const cycle = Math.floor(score / 700);
		isNight = cycle % 2 === 1;

		// Âm thanh chúc mừng mỗi 100 điểm
		if (score > 0 && score % 100 === 0 && score !== lastMilestone) {
			lastMilestone = score;
			soundMilestone();
			blinkScoreTimer = 60; // Nhấp nháy số điểm
		}

		if (blinkScoreTimer > 0) blinkScoreTimer--;

		// Cập nhật bước chân T-Rex
		dino.legTimer++;
		if (dino.legTimer > Math.max(3, 8 - Math.floor(speed * 0.4))) {
			dino.legFrame = dino.legFrame === 0 ? 1 : 0;
			dino.legTimer = 0;
		}

		// Trọng lực cho Dino
		if (dino.isJumping) {
			dino.y += dino.vy;
			dino.vy += GRAVITY;

			const targetGround = dino.isDucking ? GROUND_Y - 30 : GROUND_Y - 47;
			if (dino.y >= targetGround) {
				dino.y = targetGround;
				dino.vy = 0;
				dino.isJumping = false;
			}
		}

		// Cập nhật chướng ngại vật
		for (let i = obstacles.length - 1; i >= 0; i--) {
			const obs = obstacles[i];
			obs.x -= speed;

			if (obs.type === 'bird') {
				obs.birdTimer = (obs.birdTimer || 0) + 1;
				if (obs.birdTimer > 8) {
					obs.birdFrame = obs.birdFrame === 0 ? 1 : 0;
					obs.birdTimer = 0;
				}
			}

			// Kiểm tra va chạm
			if (checkCollision(obs)) {
				gameOver = true;
				soundDie();
				if (score > highScore) {
					highScore = score;
					try {
						localStorage.setItem('chrome_dino_hi_score', String(highScore));
					} catch (_) {}
				}
				return;
			}

			// Xóa chướng ngại vật đi qua màn hình
			if (obs.x + obs.width < -20) {
				obstacles.splice(i, 1);
			}
		}

		// Sinh chướng ngại vật tiếp theo
		nextObstacleDistance -= speed;
		if (nextObstacleDistance <= 0) {
			spawnObstacle();
		}

		// Cập nhật mây
		for (const cloud of clouds) {
			cloud.x -= cloud.speed;
			if (cloud.x < -80) {
				cloud.x = CANVAS_WIDTH + Math.random() * 100;
				cloud.y = 30 + Math.random() * 60;
			}
		}

		// Cập nhật vết gập ghềnh trên mặt đất
		for (const bump of groundBumps) {
			bump.x -= speed;
			if (bump.x < -20) {
				bump.x = CANVAS_WIDTH + Math.random() * 30;
			}
		}
	}

	function render() {
		if (!ctx || !canvas) return;

		// Màu nền ngày / đêm
		ctx.fillStyle = isNight ? '#1e1e24' : '#f7f7f7';
		ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

		const color = getColor();
		const cloudColor = isNight ? 'rgba(255, 255, 255, 0.25)' : 'rgba(83, 83, 83, 0.35)';

		// 1. Vẽ Sao & Mặt trăng nếu ban đêm
		if (isNight) {
			drawMoon(ctx, CANVAS_WIDTH - 90, 50);
			ctx.fillStyle = '#ffffff';
			for (const star of stars) {
				ctx.globalAlpha = star.alpha;
				ctx.fillRect(star.x, star.y, star.size, star.size);
			}
			ctx.globalAlpha = 1;
		}

		// 2. Vẽ Mây
		for (const cloud of clouds) {
			drawCloud(ctx, cloud.x, cloud.y, cloudColor);
		}

		// 3. Vẽ Mặt đất (Ground Line)
		ctx.fillStyle = color;
		ctx.fillRect(0, GROUND_Y, CANVAS_WIDTH, 1.5);

		// Vết lồi lõm sỏi đá trên mặt đất
		for (const bump of groundBumps) {
			if (bump.type === 0) {
				ctx.fillRect(bump.x, GROUND_Y + 4, 3, 1.5);
			} else if (bump.type === 1) {
				ctx.fillRect(bump.x, GROUND_Y + 6, 5, 1.5);
				ctx.fillRect(bump.x + 2, GROUND_Y + 8, 3, 1.5);
			} else {
				ctx.fillRect(bump.x, GROUND_Y + 3, 2, 1.5);
			}
		}

		// 4. Vẽ Chướng ngại vật
		for (const obs of obstacles) {
			if (obs.type === 'cactus_small') {
				drawSmallCactus(ctx, obs.x, obs.y, color);
			} else if (obs.type === 'cactus_large') {
				drawLargeCactus(ctx, obs.x, obs.y, color);
			} else if (obs.type === 'cactus_group') {
				drawCactusGroup(ctx, obs.x, obs.y, color);
			} else if (obs.type === 'bird') {
				drawBird(ctx, obs.x, obs.y, color, obs.birdFrame || 0);
			}
		}

		// 5. Vẽ T-Rex
		drawDino(ctx);

		// 6. Vẽ Điểm số kiểu 7-Segment (Format 5 chữ số: HI 00500 00120)
		const formatScore = (num: number) => String(num).padStart(5, '0');
		ctx.font = '700 16px "JetBrains Mono", monospace';
		ctx.fillStyle = color;
		ctx.textAlign = 'right';

		const displayScore = formatScore(score);
		const displayHi = formatScore(highScore);

		if (highScore > 0) {
			ctx.fillStyle = isNight ? '#888899' : '#737373';
			ctx.fillText(`HI ${displayHi}  `, CANVAS_WIDTH - 90, 32);
		}

		// Hiệu ứng chớp tắt khi đạt mốc điểm tròn
		if (blinkScoreTimer === 0 || Math.floor(blinkScoreTimer / 8) % 2 === 0) {
			ctx.fillStyle = color;
			ctx.fillText(displayScore, CANVAS_WIDTH - 24, 32);
		}
	}

	function loop() {
		update();
		render();
		animationFrameId = requestAnimationFrame(loop);
	}

	onMount(() => {
		if (canvas) {
			ctx = canvas.getContext('2d');
			// Scale độ phân giải sắc nét
			canvas.width = CANVAS_WIDTH;
			canvas.height = CANVAS_HEIGHT;
		}

		try {
			const savedHi = localStorage.getItem('chrome_dino_hi_score');
			if (savedHi) highScore = parseInt(savedHi, 10) || 0;
		} catch (_) {}

		initGame();
		render();

		window.addEventListener('keydown', handleKeyDown);
		window.addEventListener('keyup', handleKeyUp);

		animationFrameId = requestAnimationFrame(loop);

		return () => {
			cancelAnimationFrame(animationFrameId);
			window.removeEventListener('keydown', handleKeyDown);
			window.removeEventListener('keyup', handleKeyUp);
		};
	});
</script>

<div class="dino-page" class:dark-mode={isNight}>
	<!-- Header điều hướng chuẩn Portfolio -->
	<header class="top-nav">
		<div class="nav-left">
			<a href={resolve('/')} class="dino-home-btn" title="Quay về trang chủ">
				<span class="btn-arrow" aria-hidden="true">◄</span>
				<span class="btn-home-icon" aria-hidden="true">
					<svg viewBox="0 0 24 24" fill="currentColor" width="14" height="14">
						<path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z" />
					</svg>
				</span>
				<span class="btn-text">TRANG CHỦ</span>
			</a>
		</div>

		<div class="nav-center">
			<span class="pixel-brand">T-REX RUNNER</span>
			<span class="offline-tag">{isNight ? '🌙 NIGHT' : '☀️ DAY'}</span>
		</div>

		<div class="nav-right">
			<button
				type="button"
				class="icon-btn"
				class:muted={!soundOn}
				onclick={() => (soundOn = !soundOn)}
				title={soundOn ? 'Tắt âm thanh' : 'Bật âm thanh'}
				aria-label="Bật tắt âm thanh"
			>
				{soundOn ? '🔊' : '🔇'}
			</button>
		</div>
	</header>

	<!-- Khung cảnh trò chơi Khủng Long Mất Mạng kinh điển -->
	<main class="arcade-stage">
		<!-- Màn hình thông báo "No Internet" đậm chất Chrome -->
		{#if !gameStarted}
			<div class="offline-prompt">
				<div class="dino-icon-svg" aria-hidden="true">🦖</div>
				<h1 class="prompt-title">No internet</h1>
				<p class="prompt-desc">Try:</p>
				<ul class="prompt-tips">
					<li>Checking the network cables, modem, and router</li>
					<li>Reconnecting to Wi-Fi</li>
				</ul>
				<p class="err-code">ERR_INTERNET_DISCONNECTED</p>
				<p class="hint-start">
					<kbd>SPACE</kbd> hoặc chạm vào màn hình để bắt đầu chơi
				</p>
			</div>
		{/if}

		<!-- Khung Canvas chơi game -->
		<div
			class="canvas-box"
			role="button"
			tabindex="0"
			onclick={jump}
			onkeydown={(e) => {
				if (e.code === 'Space') jump();
			}}
		>
			<canvas bind:this={canvas}></canvas>

			<!-- Overlay Game Over phong cách Pixel Retro -->
			{#if gameOver}
				<div class="gameover-overlay">
					<p class="gameover-text">G A M E &nbsp; O V E R</p>
					<button
						type="button"
						class="restart-btn"
						onclick={(e) => {
							e.stopPropagation();
							initGame();
							gameStarted = true;
						}}
						title="Chơi lại"
						aria-label="Chơi lại"
					>
						<svg class="reload-icon" viewBox="0 0 24 24" fill="currentColor">
							<path
								d="M17.65 6.35C16.2 4.9 14.21 4 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08c-.82 2.33-3.04 4-5.65 4-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"
							/>
						</svg>
					</button>
				</div>
			{/if}
		</div>

		<!-- Bảng điều khiển cảm ứng dành cho Mobile & Tablet -->
		<div class="mobile-touch-bar">
			<button
				type="button"
				class="touch-action-btn duck-btn"
				onpointerdown={(e) => {
					e.preventDefault();
					duck(true);
				}}
				onpointerup={(e) => {
					e.preventDefault();
					duck(false);
				}}
				onpointerleave={() => duck(false)}
				aria-label="Cúi người né chim"
			>
				<span class="btn-symbol">▼</span>
				<span class="btn-text">CÚI</span>
			</button>

			<button
				type="button"
				class="touch-action-btn jump-btn"
				onpointerdown={(e) => {
					e.preventDefault();
					jump();
				}}
				aria-label="Nhảy qua chướng ngại vật"
			>
				<span class="btn-symbol">▲</span>
				<span class="btn-text">NHẢY</span>
			</button>
		</div>

		<!-- Hướng dẫn điều khiển bàn phím -->
		<footer class="instruction-bar">
			<div class="inst-item">
				<kbd>SPACE</kbd> / <kbd>↑</kbd> <span>Nhảy</span>
			</div>
			<div class="inst-item">
				<kbd>↓</kbd> <span>Cúi người né chim</span>
			</div>
			<div class="inst-item">
				<kbd>P</kbd> <span>Tạm dừng</span>
			</div>
		</footer>
	</main>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@500;600;700&family=JetBrains+Mono:wght@600;700;800&display=swap');

	.dino-page {
		position: fixed;
		inset: 0;
		width: 100vw;
		height: 100vh;
		background-color: #f7f7f7;
		display: flex;
		flex-direction: column;
		font-family: 'Plus Jakarta Sans', sans-serif;
		color: #535353;
		overflow: hidden;
		user-select: none;
		-webkit-user-select: none;
		transition: background-color 0.4s ease, color 0.4s ease;
	}

	.dino-page.dark-mode {
		background-color: #1e1e24;
		color: #e2e8f0;
	}

	/* =================================================
	   TOP NAV (Không che giao diện)
	   ================================================= */
	.top-nav {
		width: 100%;
		height: 56px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 1.25rem;
		background: rgba(255, 255, 255, 0.7);
		border-bottom: 1px solid rgba(0, 0, 0, 0.08);
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
		z-index: 50;
		box-sizing: border-box;
		transition: background 0.4s ease, border-color 0.4s ease;
	}

	.dark-mode .top-nav {
		background: rgba(24, 24, 32, 0.75);
		border-bottom: 1px solid rgba(255, 255, 255, 0.08);
	}

	.nav-left {
		display: flex;
		align-items: center;
	}

	/* Nút BackHome phong cách Pixel Retro Arcade cho Chrome Dino */
	.dino-home-btn {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.38rem 0.85rem;
		border-radius: 6px;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.78rem;
		font-weight: 800;
		letter-spacing: 0.08em;
		text-decoration: none;
		color: #535353;
		background: #ffffff;
		border: 2px solid #535353;
		box-shadow: 2px 2px 0px #535353;
		cursor: pointer;
		transition: all 0.15s cubic-bezier(0.16, 1, 0.3, 1);
	}

	.dino-home-btn:hover {
		background: #535353;
		color: #ffffff;
		box-shadow: 3px 3px 0px rgba(0, 0, 0, 0.4);
		transform: translate(-1px, -1px);
	}

	.dino-home-btn:hover .btn-arrow {
		transform: translateX(-2px);
	}

	.dino-home-btn:active {
		transform: translate(1px, 1px);
		box-shadow: 0px 0px 0px #535353;
	}

	.btn-arrow {
		font-size: 0.75rem;
		transition: transform 0.15s ease;
	}

	.btn-home-icon {
		display: flex;
		align-items: center;
	}

	.dark-mode .dino-home-btn {
		color: #e2e8f0;
		background: #252530;
		border-color: #e2e8f0;
		box-shadow: 2px 2px 0px #e2e8f0;
	}

	.dark-mode .dino-home-btn:hover {
		background: #e2e8f0;
		color: #1e1e24;
		box-shadow: 3px 3px 0px rgba(255, 255, 255, 0.35);
	}

	.dark-mode .dino-home-btn:active {
		box-shadow: 0px 0px 0px #e2e8f0;
	}

	.nav-center {
		display: flex;
		align-items: center;
		gap: 0.6rem;
	}

	.pixel-brand {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.85rem;
		font-weight: 800;
		letter-spacing: 0.14em;
	}

	.offline-tag {
		font-size: 0.65rem;
		font-weight: 700;
		padding: 0.15rem 0.45rem;
		border-radius: 999px;
		background: rgba(0, 0, 0, 0.06);
		font-family: 'JetBrains Mono', monospace;
	}

	.dark-mode .offline-tag {
		background: rgba(255, 255, 255, 0.12);
	}

	.icon-btn {
		width: 36px;
		height: 36px;
		display: flex;
		align-items: center;
		justify-content: center;
		background: rgba(0, 0, 0, 0.05);
		border: 1px solid rgba(0, 0, 0, 0.1);
		border-radius: 8px;
		color: inherit;
		cursor: pointer;
		font-size: 0.95rem;
		transition: all 0.2s ease;
	}

	.dark-mode .icon-btn {
		background: rgba(255, 255, 255, 0.08);
		border-color: rgba(255, 255, 255, 0.15);
	}

	.icon-btn.muted {
		opacity: 0.45;
	}

	/* =================================================
	   KHUNG CHƠI CHÍNH
	   ================================================= */
	.arcade-stage {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 1rem 1.5rem;
		box-sizing: border-box;
		position: relative;
	}

	/* Màn hình No Internet nguyên bản Chrome */
	.offline-prompt {
		max-width: 580px;
		width: 100%;
		margin-bottom: 1.25rem;
		animation: fadeIn 0.3s ease;
	}

	.dino-icon-svg {
		font-size: 2.2rem;
		margin-bottom: 0.5rem;
	}

	.prompt-title {
		font-size: 1.5rem;
		font-weight: 600;
		margin: 0 0 0.5rem;
	}

	.prompt-desc {
		margin: 0 0 0.35rem;
		font-size: 0.9rem;
	}

	.prompt-tips {
		margin: 0 0 0.75rem 1.25rem;
		padding: 0;
		font-size: 0.85rem;
		line-height: 1.6;
		color: #737373;
	}

	.dark-mode .prompt-tips {
		color: #94a3b8;
	}

	.err-code {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.75rem;
		color: #888888;
		margin: 0 0 0.75rem;
	}

	.hint-start {
		font-size: 0.85rem;
		font-weight: 600;
		color: #10b981;
		margin: 0;
		display: flex;
		align-items: center;
		gap: 0.4rem;
	}

	.hint-start kbd {
		background: rgba(0, 0, 0, 0.08);
		border: 1px solid rgba(0, 0, 0, 0.15);
		border-radius: 4px;
		padding: 0.15rem 0.5rem;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.75rem;
		color: inherit;
	}

	.dark-mode .hint-start kbd {
		background: rgba(255, 255, 255, 0.1);
		border-color: rgba(255, 255, 255, 0.2);
	}

	/* Khung Canvas */
	.canvas-box {
		position: relative;
		width: 100%;
		max-width: 800px;
		aspect-ratio: 800 / 280;
		border-radius: 12px;
		overflow: hidden;
		cursor: pointer;
		outline: none;
		box-shadow: 0 6px 24px rgba(0, 0, 0, 0.06);
	}

	.dark-mode .canvas-box {
		box-shadow: 0 8px 30px rgba(0, 0, 0, 0.4);
	}

	canvas {
		width: 100%;
		height: 100%;
		display: block;
		image-rendering: pixelated;
		image-rendering: crisp-edges;
	}

	/* Overlay Game Over */
	.gameover-overlay {
		position: absolute;
		top: 38%;
		left: 50%;
		transform: translate(-50%, -50%);
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.1rem;
		z-index: 10;
		animation: popIn 0.25s cubic-bezier(0.16, 1, 0.3, 1);
	}

	.gameover-text {
		font-family: 'JetBrains Mono', monospace;
		font-size: clamp(1.1rem, 3.5vw, 1.45rem);
		font-weight: 800;
		letter-spacing: 0.35em;
		margin: 0;
	}

	.restart-btn {
		width: 48px;
		height: 48px;
		border-radius: 50%;
		background: rgba(0, 0, 0, 0.07);
		border: 1px solid rgba(0, 0, 0, 0.12);
		color: inherit;
		display: flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
		transition: transform 0.2s ease, background 0.2s ease;
	}

	.dark-mode .restart-btn {
		background: rgba(255, 255, 255, 0.1);
		border-color: rgba(255, 255, 255, 0.2);
	}

	.restart-btn:hover {
		transform: scale(1.1) rotate(45deg);
		background: rgba(0, 0, 0, 0.14);
	}

	.reload-icon {
		width: 26px;
		height: 26px;
	}

	/* Bàn phím cảm ứng trên Mobile */
	.mobile-touch-bar {
		display: none;
		width: 100%;
		max-width: 400px;
		justify-content: space-between;
		gap: 1.5rem;
		margin-top: 1rem;
	}

	.touch-action-btn {
		flex: 1;
		height: 52px;
		border-radius: 14px;
		background: rgba(0, 0, 0, 0.06);
		border: 1px solid rgba(0, 0, 0, 0.12);
		color: inherit;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.9rem;
		font-weight: 800;
		cursor: pointer;
		touch-action: manipulation;
		transition: transform 0.1s ease, background 0.1s ease;
	}

	.dark-mode .touch-action-btn {
		background: rgba(255, 255, 255, 0.08);
		border-color: rgba(255, 255, 255, 0.15);
	}

	.touch-action-btn:active {
		transform: scale(0.94);
		background: rgba(16, 185, 129, 0.2);
	}

	.btn-symbol {
		font-size: 1.15rem;
	}

	/* Hướng dẫn phím */
	.instruction-bar {
		display: flex;
		align-items: center;
		gap: 1.5rem;
		margin-top: 1.25rem;
		font-size: 0.78rem;
		color: #737373;
	}

	.dark-mode .instruction-bar {
		color: #94a3b8;
	}

	.inst-item {
		display: flex;
		align-items: center;
		gap: 0.4rem;
	}

	.inst-item kbd {
		background: rgba(0, 0, 0, 0.06);
		border: 1px solid rgba(0, 0, 0, 0.12);
		border-radius: 4px;
		padding: 0.1rem 0.45rem;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.72rem;
		font-weight: 700;
		color: inherit;
	}

	.dark-mode .inst-item kbd {
		background: rgba(255, 255, 255, 0.08);
		border-color: rgba(255, 255, 255, 0.16);
	}

	@keyframes fadeIn {
		from { opacity: 0; transform: translateY(-8px); }
		to { opacity: 1; transform: translateY(0); }
	}

	@keyframes popIn {
		from { opacity: 0; transform: translate(-50%, -40%) scale(0.92); }
		to { opacity: 1; transform: translate(-50%, -50%) scale(1); }
	}

	@media (max-width: 768px) {
		.mobile-touch-bar {
			display: flex;
		}

		.instruction-bar {
			display: none;
		}

		.offline-prompt {
			margin-bottom: 0.8rem;
		}

		.prompt-title {
			font-size: 1.25rem;
		}

		.prompt-tips {
			display: none;
		}

		.prompt-desc {
			display: none;
		}
	}
</style>

