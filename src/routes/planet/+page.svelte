<svelte:options runes={true} />

<script lang="ts">
	import { onMount } from 'svelte';
	import BackHome from '$lib/components/BackHome.svelte';

	// Chu kỳ quay 1 vòng quỹ đạo hoàn chỉnh (ms)
	const PERIOD = 7500;
	// Góc nghiêng của mặt phẳng quỹ đạo so với phương ngang
	const TILT_DEG = -15;
	const TILT_RAD = (TILT_DEG * Math.PI) / 180;

	let angle = $state(0);
	let sceneWidth = $state(480);

	// Tự động tính toán kích thước theo tỷ lệ màn hình (hoàn toàn responsive trên mobile & desktop)
	let stageSize = $derived(Math.max(280, Math.min(sceneWidth, 480)));
	let halfSize = $derived(stageSize / 2);
	let planetRadius = $derived(stageSize * 0.21); // Bán kính mặt trăng/hành tinh lớn
	let rx = $derived(stageSize * 0.45);          // Bán trục lớn elip (luôn rộng hơn 2.1x mặt trăng, không bị kẹt)
	let ry = $derived(stageSize * 0.16);          // Bán trục nhỏ elip

	// Tọa độ elip nghiêng chính xác
	let x0 = $derived(Math.cos(angle));
	let y0 = $derived(Math.sin(angle));

	// Tọa độ quay nghiêng 3D
	let sx = $derived((x0 * Math.cos(TILT_RAD) * rx) - (y0 * Math.sin(TILT_RAD) * ry));
	let sy = $derived((x0 * Math.sin(TILT_RAD) * rx) + (y0 * Math.cos(TILT_RAD) * ry));

	// Chiều sâu Z: y0 > 0 là phía trước (gần người xem), y0 < 0 là phía sau (xa người xem)
	let isFront = $derived(y0 >= 0);
	// Phóng to nhẹ khi ở gần, thu nhỏ khi ở xa (chuẩn vật lý quang học viễn cận)
	let depthScale = $derived(1 + 0.18 * y0);
	// Độ sáng biến thiên liên tục mượt mà: duy trì bề mặt phát sáng chân thực, không bị hóa đen phi vật lý
	let depthBrightness = $derived(0.92 + 0.08 * y0);

	// Sao nền tĩnh ngẫu nhiên nhẹ nhàng
	const bgStars = [
		{ t: '12%', l: '18%', s: '2px', o: 0.6 },
		{ t: '22%', l: '82%', s: '1.5px', o: 0.5 },
		{ t: '78%', l: '14%', s: '2px', o: 0.7 },
		{ t: '85%', l: '75%', s: '1px', o: 0.4 },
		{ t: '15%', l: '60%', s: '2.5px', o: 0.8 },
		{ t: '65%', l: '88%', s: '1.5px', o: 0.5 },
		{ t: '40%', l: '8%', s: '1px', o: 0.4 },
		{ t: '90%', l: '45%', s: '2px', o: 0.6 }
	];

	onMount(() => {
		let rafId: number;
		let startTime: number | null = null;

		function tick(ts: number) {
			if (startTime === null) startTime = ts;
			angle = (((ts - startTime) / PERIOD) * Math.PI * 2) % (Math.PI * 2);
			rafId = requestAnimationFrame(tick);
		}

		rafId = requestAnimationFrame(tick);
		return () => cancelAnimationFrame(rafId);
	});
</script>

<svelte:head>
	<title>Animate Planet | Quỹ Đạo Chuyển Động 3D Chuẩn Vật Lý</title>
	<meta
		name="description"
		content="Mô phỏng quỹ đạo hành tinh xoay quanh mặt trăng với chuyển động 3D chuẩn xác, không nháy hình, không che khuất sai vật lý."
	/>
</svelte:head>

<div class="wrapper">
	<!-- Thanh điều hướng trên cùng (Không che tiêu đề) -->
	<header class="top-nav">
		<div class="nav-left">
			<BackHome label="Trang chủ" />
		</div>
		<div class="nav-title">
			<span class="pulse-dot"></span>
			<span class="title-text">ORBITAL MOTION SIMULATION</span>
		</div>
	</header>

	<!-- Bầu trời sao nền vi tế -->
	<div class="starfield" aria-hidden="true">
		{#each bgStars as star, i (i)}
			<span
				class="star"
				style="top: {star.t}; left: {star.l}; width: {star.s}; height: {star.s}; opacity: {star.o};"
			></span>
		{/each}
	</div>

	<!-- Không gian mô phỏng quỹ đạo -->
	<main class="scene-stage">
		<div class="scene" bind:clientWidth={sceneWidth}>
			<!-- Vạch quỹ đạo elip (Khớp 100% từng pixel với đường đi của hành tinh con) -->
			<svg
				class="orbit-svg"
				viewBox="{-halfSize} {-halfSize} {stageSize} {stageSize}"
				style="width: {stageSize}px; height: {stageSize}px;"
				xmlns="http://www.w3.org/2000/svg"
				aria-hidden="true"
			>
				<ellipse
					cx="0"
					cy="0"
					{rx}
					{ry}
					transform="rotate({TILT_DEG})"
					fill="none"
					stroke="rgba(255, 255, 255, 0.14)"
					stroke-width="1.2"
					stroke-dasharray="5 7"
				/>
			</svg>

			<!-- Hành tinh mẹ (Mặt trăng / Moon - Nằm ở z-index 5) -->
			<div class="planet" style="width: {planetRadius * 2}px; height: {planetRadius * 2}px;">
				<div class="planet-shine"></div>
				<!-- Các hố va chạm (Craters) đổ bóng theo nguồn sáng góc trên bên trái -->
				<div class="planet-crater crater-1"></div>
				<div class="planet-crater crater-2"></div>
				<div class="planet-crater crater-3"></div>
				<div class="planet-crater crater-4"></div>
			</div>

			<!-- 
				Hành tinh con duy nhất (Single DOM Element):
				- Ở trước mặt trăng: z-index = 10, scale lớn hơn, sáng rực
				- Ở sau mặt trăng: z-index = 2, mặt trăng tự nhiên che khuất, KHÔNG bị unmount nháy hình, KHÔNG bị đen ngòm
			-->
			<div
				class="satellite"
				style="
					transform: translate(calc(-50% + {sx}px), calc(-50% + {sy}px)) scale({depthScale});
					z-index: {isFront ? 10 : 2};
					filter: brightness({depthBrightness});
				"
			>
				<div class="satellite-surface"></div>
				<div class="satellite-halo"></div>
			</div>
		</div>
	</main>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@500;600;700&family=JetBrains+Mono:wght@500;700&display=swap');

	.wrapper {
		position: fixed;
		inset: 0;
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		background: radial-gradient(ellipse at 50% 40%, #151838 0%, #0d0f22 55%, #060711 100%);
		overflow: hidden;
		user-select: none;
		font-family: 'Plus Jakarta Sans', sans-serif;
		color: #ffffff;
	}

	/* =================================================
	   TOP NAV (Không che tiêu đề)
	   ================================================= */
	.top-nav {
		height: 60px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 1.5rem;
		background: rgba(10, 12, 26, 0.7);
		border-bottom: 1px solid rgba(255, 255, 255, 0.08);
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
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
	}

	.nav-title {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.75rem;
		letter-spacing: 0.18em;
		color: #94a3b8;
	}

	.pulse-dot {
		width: 6px;
		height: 6px;
		border-radius: 50%;
		background-color: #38bdf8;
		box-shadow: 0 0 8px #38bdf8;
	}

	/* =================================================
	   STARFIELD BACKGROUND
	   ================================================= */
	.starfield {
		position: absolute;
		inset: 0;
		pointer-events: none;
		z-index: 0;
	}

	.star {
		position: absolute;
		border-radius: 50%;
		background-color: #ffffff;
		box-shadow: 0 0 3px rgba(255, 255, 255, 0.8);
	}

	/* =================================================
	   SCENE STAGE & ORBIT
	   ================================================= */
	.scene-stage {
		flex: 1;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		padding: 1.5rem;
		box-sizing: border-box;
		z-index: 1;
	}

	.scene {
		position: relative;
		width: min(88vw, 480px);
		height: min(88vw, 480px);
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.orbit-svg {
		position: absolute;
		width: 100%;
		height: 100%;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
		pointer-events: none;
		overflow: visible;
		z-index: 1;
	}

	/* =================================================
	   MẶT TRĂNG (PLANET / MOON - z-index 5)
	   ================================================= */
	.planet {
		position: absolute;
		border-radius: 50%;
		background: radial-gradient(circle at 35% 32%, #ffffff 0%, #f1f5f9 25%, #cbd5e1 55%, #64748b 90%, #334155 100%);
		box-shadow:
			0 0 45px rgba(254, 216, 76, 0.45),
			0 0 100px rgba(232, 165, 82, 0.2),
			0 0 160px rgba(56, 189, 248, 0.12),
			inset -12px -12px 28px rgba(15, 23, 42, 0.7);
		overflow: hidden;
		z-index: 5;
		transition: width 0.2s ease, height 0.2s ease;
	}

	.planet-shine {
		position: absolute;
		top: 10%;
		left: 14%;
		width: 32%;
		height: 28%;
		border-radius: 50%;
		background: radial-gradient(circle, rgba(255, 255, 255, 0.75) 0%, transparent 70%);
		filter: blur(5px);
		pointer-events: none;
	}

	.planet-crater {
		position: absolute;
		border-radius: 50%;
		background: rgba(100, 116, 139, 0.35);
		box-shadow: 
			inset 2px 2px 4px rgba(15, 23, 42, 0.6),
			1px 1px 2px rgba(255, 255, 255, 0.25);
	}

	.crater-1 { width: 20%; height: 20%; top: 52%; left: 22%; }
	.crater-2 { width: 14%; height: 14%; top: 32%; left: 62%; }
	.crater-3 { width: 10%; height: 10%; top: 68%; left: 56%; }
	.crater-4 { width: 8%; height: 8%; top: 22%; left: 42%; }

	/* =================================================
	   HÀNH TINH CON (SATELLITE / ORBITING BODY)
	   ================================================= */
	.satellite {
		position: absolute;
		top: 50%;
		left: 50%;
		width: clamp(28px, 6vw, 44px);
		height: clamp(28px, 6vw, 44px);
		border-radius: 50%;
		will-change: transform, filter;
		pointer-events: none;
		transition: filter 0.2s ease;
	}

	/* Bề mặt hình cầu luôn được chiếu sáng tự nhiên, không bị hóa đen phi vật lý */
	.satellite-surface {
		position: absolute;
		inset: 0;
		border-radius: 50%;
		background: radial-gradient(circle at 35% 30%, #fffdfa 0%, #fed7aa 35%, #f97316 75%, #9a3412 100%);
		box-shadow: 
			inset -2px -2px 6px rgba(67, 20, 7, 0.7),
			0 0 16px rgba(251, 146, 60, 0.5);
	}

	.satellite-halo {
		position: absolute;
		inset: -2px;
		border-radius: 50%;
		background: radial-gradient(circle, rgba(254, 215, 170, 0.4) 0%, transparent 75%);
		filter: blur(3px);
	}

	/* =================================================
	   RESPONSIVE
	   ================================================= */
	@media (max-width: 640px) {
		.top-nav {
			padding: 0 1rem;
		}

		.title-text {
			font-size: 0.65rem;
			letter-spacing: 0.1em;
		}
	}
</style>
