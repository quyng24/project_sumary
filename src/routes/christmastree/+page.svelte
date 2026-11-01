<svelte:options runes={true} />

<script lang="ts">
	import { onMount } from 'svelte';
	import merryChristmasTrack from '$lib/assets/merry-christmas.mp3';
	import BackHome from '$lib/components/BackHome.svelte';

	// Cấu hình cây thông số (Matrix Binary Tree)
	const TREE_HEIGHT = 15;
	const TREE_REFRESH_INTERVAL_MS = 800;
	const currentYear = new Date().getFullYear();

	interface TreeCell {
		digit: '0' | '1';
		color: string;
		glow: boolean;
	}

	interface SnowflakeConfig {
		id: number;
		left: number;
		size: number;
		duration: number;
		delay: number;
		drift: number;
		opacity: number;
	}

	interface HolidayGift {
		id: number;
		label: string;
		color: string;
		ribbon: string;
		wish: string;
		opened: boolean;
	}

	let treeRows = $state<TreeCell[][]>([]);
	let isPlaying = $state(false);
	let activeWish = $state<string | null>(null);
	let audioEl: HTMLAudioElement | null = null;

	// Tạo trước 38 bông tuyết cố định, chuyển động 100% bằng GPU CSS keyframes (không tốn CPU/RAM, không giật lag)
	const snowflakes: SnowflakeConfig[] = Array.from({ length: 38 }, (_, i) => ({
		id: i,
		left: Math.round((i * 2.65 + (i % 3) * 1.5) % 98),
		size: (i % 5) * 1.2 + 3,
		duration: 6 + (i % 6) * 1.5,
		delay: (i % 7) * 1.2,
		drift: ((i % 4) - 1.5) * 40,
		opacity: 0.35 + ((i % 5) * 0.12)
	}));

	// 3 Hộp quà Giáng sinh tương tác đặt dưới gốc cây thông
	let gifts = $state<HolidayGift[]>([
		{
			id: 1,
			label: 'An Lành',
			color: '#059669', // Emerald
			ribbon: '#fbbf24', // Vàng hoàng kim
			wish: 'Chúc bạn và gia đình một mùa Giáng sinh ngập tràn an lành, ấm áp và yêu thương! 🎄✨',
			opened: false
		},
		{
			id: 2,
			label: 'Thành Công',
			color: '#dc2626', // Ruby
			ribbon: '#f8fafc', // Trắng bạc tuyết
			wish: `Chào đón năm mới ${currentYear}! Chúc mọi dự án bứt phá, code sạch mượt mà, zero bug! 💻🚀`,
			opened: false
		},
		{
			id: 3,
			label: 'Hạnh Phúc',
			color: '#2563eb', // Sapphire
			ribbon: '#fbbf24', // Vàng hoàng kim
			wish: 'Chúc bạn luôn dồi dào năng lượng sáng tạo, vững bước trên hành trình đam mê! 🌟⭐',
			opened: false
		}
	]);

	function generateTree(): void {
		const rows: TreeCell[][] = [];
		for (let i = 0; i < TREE_HEIGHT; i++) {
			const row: TreeCell[] = [];
			for (let j = 0; j < 2 * i + 1; j++) {
				const rand = Math.random();
				const isOrnament = rand < 0.15;
				let color = '#10b981'; // Xanh lá cây thông

				if (isOrnament) {
					if (rand < 0.05) color = '#ef4444'; // Quả châu đỏ ruby
					else if (rand < 0.1) color = '#fbbf24'; // Quả chuông vàng kim
					else color = '#38bdf8'; // Đèn led xanh băng tuyết
				}

				row.push({
					digit: Math.random() < 0.5 ? '0' : '1',
					color,
					glow: isOrnament
				});
			}
			rows.push(row);
		}
		treeRows = rows;
	}

	function handleGiftClick(gift: HolidayGift): void {
		gifts = gifts.map((g) => (g.id === gift.id ? { ...g, opened: true } : g));
		activeWish = gift.wish;
	}

	async function toggleAudio(): Promise<void> {
		if (!audioEl) return;
		if (isPlaying) {
			audioEl.pause();
			isPlaying = false;
		} else {
			try {
				await audioEl.play();
				isPlaying = true;
			} catch (err) {
				console.error('Audio playback failed:', err);
				isPlaying = false;
			}
		}
	}

	onMount(() => {
		generateTree();
		const treeTimer = setInterval(generateTree, TREE_REFRESH_INTERVAL_MS);
		return () => clearInterval(treeTimer);
	});
</script>

<svelte:head>
	<title>Digital Christmas Tree {currentYear} | Cây Thông Số & Lời Chúc Giáng Sinh</title>
	<meta
		name="description"
		content="Trải nghiệm cây thông giáng sinh số phong cách ma trận với âm nhạc lễ hội và hộp quà tương tác."
	/>
</svelte:head>

<div class="christmas-page">
	<!-- Thanh điều hướng phía trên -->
	<header class="top-nav">
		<div class="nav-left">
			<BackHome label="Trang chủ" />
		</div>

		<div class="nav-center">
			<span class="sparkle-icon">✦</span>
			<span class="title-text">DIGITAL MATRIX XMAS</span>
			<span class="year-badge">{currentYear}</span>
		</div>

		<div class="nav-right">
			<button
				class="music-pill"
				class:playing={isPlaying}
				onclick={toggleAudio}
				title={isPlaying ? 'Tạm dừng nhạc' : 'Bật nhạc Giáng sinh'}
				aria-label={isPlaying ? 'Tạm dừng nhạc' : 'Bật nhạc Giáng sinh'}
			>
				<div class="eq-bars" aria-hidden="true">
					<span class="bar bar-1"></span>
					<span class="bar bar-2"></span>
					<span class="bar bar-3"></span>
				</div>
				<span class="music-label">{isPlaying ? 'MERRY CHRISTMAS' : 'PLAY MUSIC'}</span>
			</button>
		</div>
	</header>

	<!-- Hiệu ứng tuyết rơi thuần GPU CSS (nhẹ, không đơ lag) -->
	<div class="snow-layer" aria-hidden="true">
		{#each snowflakes as flake (flake.id)}
			<div
				class="snowflake"
				style="
					left: {flake.left}%;
					width: {flake.size}px;
					height: {flake.size}px;
					animation-duration: {flake.duration}s;
					animation-delay: -{flake.delay}s;
					--snow-opacity: {flake.opacity};
					--snow-drift: {flake.drift}px;
				"
			></div>
		{/each}
	</div>

	<!-- Thẻ Audio chạy nhạc Giáng sinh ngầm -->
	<audio bind:this={audioEl} loop src={merryChristmasTrack}></audio>

	<!-- Không gian chính: Cây thông & Hộp quà tương tác thay cho chiếc card to cũ -->
	<main class="tree-stage">
		<!-- Ánh hào quang tỏa ra từ tâm cây thông -->
		<div class="ambient-glow" aria-hidden="true"></div>

		<!-- Khối Cây Thông Nhị Phân -->
		<div class="tree-container">
			<!-- Ngôi sao vàng đỉnh cây thông -->
			<div class="star-wrapper">
				<div class="star-halo" aria-hidden="true"></div>
				<div class="star-glyph">★</div>
			</div>

			<!-- Thân cây cấu thành từ chuỗi ký tự số 0/1 đổi màu -->
			<div class="binary-canopy">
				{#each treeRows as row, i (i)}
					<div class="tree-row">
						{#each row as cell, j (j)}
							<span class="tree-cell" class:glow={cell.glow} style="color: {cell.color};">
								{cell.digit}
							</span>
						{/each}
					</div>
				{/each}
			</div>

			<!-- Gốc cây & Chân đế vững chãi -->
			<div class="tree-base">
				<div class="trunk"></div>
				<div class="pedestal-ring"></div>
			</div>
		</div>

		<!-- 
			Khu vực Hộp Quà Giáng Sinh Tương Tác:
			Thay thế hoàn toàn chiếc thẻ to chữ nhật thô kệch cũ bằng trải nghiệm tinh tế, 
			thuận mắt và mang đậm tinh thần lễ hội.
		-->
		<section class="presents-section" aria-label="Hộp quà Giáng sinh">
			<div class="gifts-row">
				{#each gifts as gift (gift.id)}
					<button
						type="button"
						class="gift-box"
						class:opened={gift.opened}
						style="--box-color: {gift.color}; --ribbon-color: {gift.ribbon};"
						onclick={() => handleGiftClick(gift)}
						title="Nhấn để mở lời chúc: {gift.label}"
					>
						<div class="gift-lid">
							<div class="lid-bow"></div>
						</div>
						<div class="gift-body">
							<div class="ribbon-v"></div>
							<div class="ribbon-h"></div>
						</div>
						<span class="gift-tag">{gift.label}</span>
					</button>
				{/each}
			</div>

			<!-- Thông điệp lời chúc hiện lên khi mở quà -->
			{#if activeWish}
				<div class="wish-toast" role="status">
					<div class="wish-inner">
						<span class="wish-sparkle">✨</span>
						<p class="wish-message">{activeWish}</p>
						<button
							type="button"
							class="wish-close"
							onclick={() => (activeWish = null)}
							aria-label="Đóng lời chúc"
						>
							✕
						</button>
					</div>
				</div>
			{:else}
				<p class="gifts-hint">
					<span class="hint-sparkle">🎁</span> Chạm vào các hộp quà dưới gốc thông để mở lời chúc Giáng sinh
				</p>
			{/if}
		</section>
	</main>

	<!-- Footer gọn gàng, thanh lịch thay cho phần chữ thô trong card cũ -->
	<footer class="footer-capsule">
		<div class="capsule-pill">
			<span class="pill-sparkle">✦</span>
			<span class="pill-title">Digital Christmas</span>
			<span class="pill-dot">•</span>
			<span class="pill-year">{currentYear}</span>
			<span class="pill-dot">•</span>
			<span class="pill-author">Nguyen Quy Coder</span>
			<span class="pill-sparkle">✦</span>
		</div>
	</footer>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@500;600;700&family=JetBrains+Mono:wght@500;700;900&display=swap');

	.christmas-page {
		position: relative;
		width: 100%;
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		align-items: center;
		background: radial-gradient(ellipse at 50% 30%, #064e3b 0%, #032b21 45%, #02120e 75%, #010605 100%);
		overflow-x: hidden;
		user-select: none;
		font-family: 'Plus Jakarta Sans', sans-serif;
		color: #ffffff;
		box-sizing: border-box;
	}

	/* =================================================
	   TOP NAV
	   ================================================= */
	.top-nav {
		width: 100%;
		height: 64px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 1.5rem;
		background: rgba(2, 18, 14, 0.75);
		border-bottom: 1px solid rgba(16, 185, 129, 0.15);
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
		padding: 0.45rem 0.95rem !important;
		font-size: 0.85rem !important;
		border-radius: 999px !important;
		background: rgba(255, 255, 255, 0.06) !important;
		border: 1px solid rgba(255, 255, 255, 0.12) !important;
		color: #ffffff !important;
		transition: all 0.2s ease !important;
	}

	:global(.top-nav .back-home:hover) {
		background: rgba(16, 185, 129, 0.2) !important;
		border-color: rgba(16, 185, 129, 0.4) !important;
	}

	.nav-center {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-family: 'JetBrains Mono', monospace;
	}

	.sparkle-icon {
		color: #fbbf24;
		font-size: 0.8rem;
		animation: spinSlow 6s linear infinite;
	}

	.title-text {
		font-size: 0.78rem;
		font-weight: 700;
		letter-spacing: 0.16em;
		color: #e2e8f0;
	}

	.year-badge {
		font-size: 0.7rem;
		padding: 0.15rem 0.45rem;
		border-radius: 999px;
		background: rgba(251, 191, 36, 0.15);
		color: #fbbf24;
		border: 1px solid rgba(251, 191, 36, 0.3);
		font-weight: 700;
	}

	.nav-right {
		display: flex;
		align-items: center;
	}

	/* Nút Music Pill sang trọng */
	.music-pill {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.45rem 1rem;
		border-radius: 999px;
		background: rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.15);
		color: #f8fafc;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.75rem;
		font-weight: 700;
		letter-spacing: 0.05em;
		cursor: pointer;
		transition: all 0.25s ease;
	}

	.music-pill:hover {
		background: rgba(16, 185, 129, 0.2);
		border-color: rgba(16, 185, 129, 0.4);
		transform: translateY(-1px);
	}

	.music-pill.playing {
		background: rgba(16, 185, 129, 0.25);
		border-color: #10b981;
		box-shadow: 0 0 16px rgba(16, 185, 129, 0.35);
		color: #a7f3d0;
	}

	.eq-bars {
		display: flex;
		align-items: flex-end;
		gap: 2px;
		height: 12px;
	}

	.eq-bars .bar {
		width: 2.5px;
		height: 100%;
		background-color: currentColor;
		border-radius: 1px;
	}

	.music-pill.playing .bar-1 {
		animation: soundWave 0.7s ease-in-out infinite alternate;
	}
	.music-pill.playing .bar-2 {
		animation: soundWave 0.5s ease-in-out 0.2s infinite alternate;
	}
	.music-pill.playing .bar-3 {
		animation: soundWave 0.8s ease-in-out 0.4s infinite alternate;
	}

	@keyframes soundWave {
		0% { height: 20%; }
		100% { height: 100%; }
	}

	@keyframes spinSlow {
		from { transform: rotate(0deg); }
		to { transform: rotate(360deg); }
	}

	/* =================================================
	   SNOW LAYER (GPU ACCELERATED)
	   ================================================= */
	.snow-layer {
		position: absolute;
		inset: 0;
		pointer-events: none;
		z-index: 10;
		overflow: hidden;
	}

	.snowflake {
		position: absolute;
		top: -20px;
		background: #ffffff;
		border-radius: 50%;
		filter: blur(0.6px);
		animation: snowFall linear infinite;
		will-change: transform, opacity;
	}

	@keyframes snowFall {
		0% {
			transform: translate3d(0, 0, 0);
			opacity: 0;
		}
		10% {
			opacity: var(--snow-opacity);
		}
		90% {
			opacity: var(--snow-opacity);
		}
		100% {
			transform: translate3d(var(--snow-drift), 105vh, 0);
			opacity: 0;
		}
	}

	/* =================================================
	   MAIN STAGE & DIGITAL TREE
	   ================================================= */
	.tree-stage {
		flex: 1;
		width: 100%;
		max-width: 900px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 1.5rem 1rem 1rem;
		box-sizing: border-box;
		position: relative;
		z-index: 20;
	}

	.ambient-glow {
		position: absolute;
		top: 35%;
		left: 50%;
		width: min(500px, 80vw);
		height: min(500px, 80vw);
		transform: translate(-50%, -50%);
		background: radial-gradient(circle, rgba(16, 185, 129, 0.16) 0%, rgba(251, 191, 36, 0.08) 40%, transparent 70%);
		border-radius: 50%;
		pointer-events: none;
		filter: blur(40px);
		z-index: -1;
	}

	.tree-container {
		display: flex;
		flex-direction: column;
		align-items: center;
		filter: drop-shadow(0 0 25px rgba(16, 185, 129, 0.25));
	}

	/* Ngôi sao đỉnh cây */
	.star-wrapper {
		position: relative;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 0.15rem;
	}

	.star-halo {
		position: absolute;
		width: 50px;
		height: 50px;
		border-radius: 50%;
		background: radial-gradient(circle, rgba(251, 191, 36, 0.6) 0%, transparent 70%);
		filter: blur(8px);
		animation: starBreathe 3s ease-in-out infinite alternate;
	}

	.star-glyph {
		font-size: 2.75rem;
		color: #fbbf24;
		text-shadow:
			0 0 12px #fbbf24,
			0 0 24px #f59e0b,
			0 0 45px rgba(251, 191, 36, 0.7);
		animation: starBreathe 3s ease-in-out infinite alternate;
		line-height: 1;
	}

	@keyframes starBreathe {
		0% {
			transform: scale(0.95);
			filter: brightness(0.9);
		}
		100% {
			transform: scale(1.1);
			filter: brightness(1.2);
		}
	}

	/* Vòm cây nhị phân (Binary Canopy) */
	.binary-canopy {
		font-family: 'JetBrains Mono', monospace;
		font-size: clamp(0.72rem, 1.8vw, 1.05rem);
		line-height: 1.15;
		letter-spacing: 0.1em;
		text-align: center;
		user-select: none;
	}

	.tree-row {
		white-space: nowrap;
	}

	.tree-cell {
		display: inline-block;
		padding: 0 1px;
		transition: color 0.4s ease, text-shadow 0.4s ease;
	}

	.tree-cell.glow {
		font-weight: 900;
		text-shadow: 0 0 10px currentColor, 0 0 18px currentColor;
		filter: brightness(1.3);
	}

	/* Gốc cây & Đế */
	.tree-base {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 4px;
	}

	.trunk {
		width: clamp(24px, 4vw, 36px);
		height: clamp(32px, 5vh, 48px);
		background: linear-gradient(180deg, #334155 0%, #1e293b 50%, #0f172a 100%);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 4px;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.6);
	}

	.pedestal-ring {
		width: clamp(70px, 12vw, 100px);
		height: 8px;
		background: radial-gradient(ellipse at center, rgba(16, 185, 129, 0.5) 0%, rgba(251, 191, 36, 0.25) 50%, transparent 80%);
		border-radius: 50%;
		filter: blur(2px);
		margin-top: -3px;
	}

	/* =================================================
	   HỘP QUÀ GIÁNG SINH TƯƠNG TÁC
	   (Thay thế hoàn toàn chiếc thẻ to cồng kềnh)
	   ================================================= */
	.presents-section {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 1.25rem;
		width: 100%;
		max-width: 500px;
	}

	.gifts-row {
		display: flex;
		align-items: flex-end;
		justify-content: center;
		gap: 1.5rem;
		padding: 0.5rem 0;
	}

	.gift-box {
		background: none;
		border: none;
		padding: 0;
		cursor: pointer;
		display: flex;
		flex-direction: column;
		align-items: center;
		transition: transform 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
		outline: none;
	}

	.gift-box:hover {
		transform: translateY(-6px) scale(1.06);
	}

	.gift-box:active {
		transform: scale(0.96);
	}

	.gift-box.opened {
		transform: translateY(-3px);
	}

	.gift-box.opened .gift-lid {
		transform: translateY(-5px) rotate(-8deg);
		transition: transform 0.3s ease;
	}

	/* Nắp hộp quà */
	.gift-lid {
		position: relative;
		width: 38px;
		height: 9px;
		background: var(--box-color);
		border-radius: 2px;
		box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
		z-index: 2;
		transition: transform 0.2s ease;
	}

	.lid-bow {
		position: absolute;
		top: -5px;
		left: 50%;
		transform: translateX(-50%);
		width: 12px;
		height: 5px;
		background: var(--ribbon-color);
		border-radius: 3px 3px 0 0;
		box-shadow: 0 -1px 4px rgba(0, 0, 0, 0.25);
	}

	/* Thân hộp quà */
	.gift-body {
		position: relative;
		width: 32px;
		height: 28px;
		background: var(--box-color);
		border-radius: 0 0 3px 3px;
		box-shadow: 0 6px 14px rgba(0, 0, 0, 0.4);
		overflow: hidden;
		margin-top: -1px;
	}

	/* Dải ruy băng nơ */
	.ribbon-v {
		position: absolute;
		top: 0;
		bottom: 0;
		left: 50%;
		transform: translateX(-50%);
		width: 6px;
		background: var(--ribbon-color);
	}

	.ribbon-h {
		position: absolute;
		left: 0;
		right: 0;
		top: 45%;
		height: 5px;
		background: var(--ribbon-color);
	}

	.gift-tag {
		margin-top: 0.45rem;
		font-size: 0.72rem;
		font-weight: 700;
		color: #cbd5e1;
		font-family: 'JetBrains Mono', monospace;
		letter-spacing: 0.05em;
		background: rgba(255, 255, 255, 0.06);
		padding: 0.15rem 0.5rem;
		border-radius: 999px;
		border: 1px solid rgba(255, 255, 255, 0.1);
		transition: all 0.2s ease;
	}

	.gift-box:hover .gift-tag {
		color: #ffffff;
		border-color: rgba(255, 255, 255, 0.3);
		background: rgba(255, 255, 255, 0.12);
	}

	/* Lời nhắc nhẹ nhàng */
	.gifts-hint {
		font-size: 0.74rem;
		color: #94a3b8;
		margin-top: 0.8rem;
		display: flex;
		align-items: center;
		gap: 0.35rem;
		text-align: center;
	}

	.hint-sparkle {
		font-size: 0.85rem;
		animation: pulse 2s infinite;
	}

	/* Toast Lời Chúc (Nhẹ nhàng, sắc nét, đẹp mắt) */
	.wish-toast {
		margin-top: 0.75rem;
		width: 100%;
		animation: slideUpFade 0.35s cubic-bezier(0.16, 1, 0.3, 1);
	}

	.wish-inner {
		display: flex;
		align-items: center;
		gap: 0.75rem;
		padding: 0.75rem 1.15rem;
		background: rgba(6, 78, 59, 0.65);
		border: 1px solid rgba(52, 211, 153, 0.4);
		border-radius: 1rem;
		backdrop-filter: blur(16px);
		-webkit-backdrop-filter: blur(16px);
		box-shadow: 0 8px 30px rgba(0, 0, 0, 0.4), 0 0 20px rgba(16, 185, 129, 0.2);
	}

	.wish-sparkle {
		font-size: 1.1rem;
		color: #fbbf24;
		flex-shrink: 0;
	}

	.wish-message {
		flex: 1;
		margin: 0;
		font-size: 0.82rem;
		line-height: 1.45;
		color: #f0fdf4;
		font-weight: 600;
	}

	.wish-close {
		background: rgba(255, 255, 255, 0.1);
		border: none;
		color: #cbd5e1;
		width: 24px;
		height: 24px;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
		font-size: 0.75rem;
		flex-shrink: 0;
		transition: all 0.2s ease;
	}

	.wish-close:hover {
		background: rgba(255, 255, 255, 0.25);
		color: #ffffff;
	}

	@keyframes slideUpFade {
		from {
			opacity: 0;
			transform: translateY(10px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	/* =================================================
	   FOOTER CAPSULE
	   ================================================= */
	.footer-capsule {
		padding: 1rem 1.5rem 1.25rem;
		display: flex;
		justify-content: center;
		z-index: 20;
		box-sizing: border-box;
	}

	.capsule-pill {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.4rem 1.25rem;
		background: rgba(2, 20, 16, 0.6);
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 999px;
		font-size: 0.75rem;
		color: #94a3b8;
		backdrop-filter: blur(10px);
		-webkit-backdrop-filter: blur(10px);
	}

	.pill-sparkle {
		color: #fbbf24;
		font-size: 0.7rem;
	}

	.pill-title {
		color: #f1f5f9;
		font-weight: 600;
	}

	.pill-dot {
		color: rgba(255, 255, 255, 0.25);
	}

	.pill-year {
		font-family: 'JetBrains Mono', monospace;
		font-weight: 700;
		color: #10b981;
	}

	.pill-author {
		color: #cbd5e1;
	}

	/* =================================================
	   RESPONSIVE DESIGN (MOBILE & TABLET)
	   ================================================= */
	@media (max-width: 640px) {
		.top-nav {
			padding: 0 0.85rem;
			height: 56px;
		}

		.title-text {
			font-size: 0.68rem;
			letter-spacing: 0.08em;
		}

		.year-badge {
			display: none;
		}

		.music-pill {
			padding: 0.35rem 0.75rem;
			font-size: 0.68rem;
		}

		.music-label {
			display: none;
		}

		.tree-stage {
			padding: 0.75rem 0.5rem 0.5rem;
		}

		.star-glyph {
			font-size: 2.2rem;
		}

		.binary-canopy {
			font-size: 0.7rem;
			line-height: 1.12;
			letter-spacing: 0.04em;
		}

		.gifts-row {
			gap: 1.1rem;
		}

		.gift-lid {
			width: 32px;
			height: 8px;
		}

		.gift-body {
			width: 28px;
			height: 24px;
		}

		.gift-tag {
			font-size: 0.65rem;
			padding: 0.1rem 0.4rem;
		}

		.capsule-pill {
			font-size: 0.68rem;
			gap: 0.4rem;
			padding: 0.35rem 0.85rem;
		}

		.pill-author {
			display: none;
		}
	}
</style>
