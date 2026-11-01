<script lang="ts">
	import { untrack } from 'svelte';

	let { value = '0' }: { value: string } = $props();

	// Lưu trữ digit hiện tại và digit tiếp theo (dùng untrack để đồng bộ khởi tạo an toàn)
	let currentDigit = $state(untrack(() => value));
	let nextDigit = $state(untrack(() => value));
	let isFlipping = $state(false);

	$effect(() => {
		const target = value;
		// Khi giá trị mới đến khác với digit đang hiển thị
		if (target !== currentDigit && !isFlipping) {
			nextDigit = target;
			isFlipping = true;
		}
	});

	function handleAnimationEnd() {
		currentDigit = nextDigit;
		isFlipping = false;
	}
</script>

<div class="digit-card" aria-label={`Digit ${currentDigit}`}>
	<!-- ==========================================
	     1. CÁC NỬA THẺ TĨNH NỀN (BACKGROUND CARDS)
	     ========================================== -->

	<!-- Nửa trên tĩnh: Hiển thị trước NEXT digit (sẽ lộ ra khi flap gập xuống) -->
	<div class="card-half top static-top">
		<span class="digit-num">{nextDigit}</span>
		<!-- Bóng tối dần tan khi lá lật rời đi -->
		<div class="shadow-layer shadow-under" class:active={isFlipping}></div>
	</div>

	<!-- Nửa dưới tĩnh: Giữ CURRENT digit (chờ lá lật úp đè lên) -->
	<div class="card-half bottom static-bottom">
		<span class="digit-num">{currentDigit}</span>
		<!-- Bóng tối dần đậm khi lá lật tiếp cận -->
		<div class="shadow-layer shadow-bottom-ambient" class:active={isFlipping}></div>
	</div>

	<!-- ==========================================
	     2. LÁ LẬT VẬT LÝ 3D (DYNAMIC FLIPPING LEAF)
	     ========================================== -->
	{#if isFlipping}
		<div class="flip-leaf" onanimationend={handleAnimationEnd}>
			<!-- Mặt trước: Nửa trên CURRENT (lật từ 0° úp về phía trước đến -90°) -->
			<div class="leaf-face front">
				<span class="digit-num">{currentDigit}</span>
				<!-- Bóng đổ khi ngả dần khỏi ánh sáng trần -->
				<div class="shadow-layer shadow-front"></div>
			</div>

			<!-- Mặt sau: Nửa dưới NEXT (từ -90° tiếp tục rơi đập xuống -180°) -->
			<div class="leaf-face back">
				<span class="digit-num">{nextDigit}</span>
				<!-- Bóng mờ dần sáng khi tiếp đất phẳng -->
				<div class="shadow-layer shadow-back"></div>
			</div>
		</div>
	{/if}

	<!-- ==========================================
	     3. CHI TIẾT CƠ KHÍ (SPLIT-FLAP DETAILS)
	     ========================================== -->
	<!-- Rãnh khe hở ngang chính giữa -->
	<div class="crease-gap"></div>

	<!-- Chốt bản lề hai bên (Side hinge notches) -->
	<div class="hinge-notch hinge-left"></div>
	<div class="hinge-notch hinge-right"></div>
</div>

<style>
	.digit-card {
		--card-w: clamp(50px, 12vw, 130px);
		--card-h: calc(var(--card-w) * 1.42);
		--radius: 8px;
		--anim-dur: 0.52s;

		position: relative;
		width: var(--card-w);
		height: var(--card-h);
		perspective: 1000px;
		-webkit-perspective: 1000px;
		border-radius: var(--radius);
		user-select: none;
		box-shadow: 
			0 12px 28px rgba(0, 0, 0, 0.65),
			0 2px 6px rgba(0, 0, 0, 0.4);
	}

	/* =================================================
	   NỬA THẺ CƠ BẢN
	   ================================================= */
	.card-half {
		position: absolute;
		left: 0;
		width: 100%;
		height: 50%;
		overflow: hidden;
		box-sizing: border-box;
	}

	.digit-num {
		position: absolute;
		left: 50%;
		width: 100%;
		height: var(--card-h);
		line-height: var(--card-h);
		text-align: center;
		transform: translateX(-50%);
		font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "Plus Jakarta Sans", "Inter", sans-serif;
		font-size: calc(var(--card-w) * 1.05);
		font-weight: 800;
		font-variant-numeric: tabular-nums;
		font-feature-settings: "tnum";
		letter-spacing: -0.02em;
		color: #f1f5f9;
		text-shadow: 0 1px 3px rgba(0, 0, 0, 0.6);
	}

	/* Nửa trên */
	.top {
		top: 0;
		border-radius: var(--radius) var(--radius) 0 0;
		background: linear-gradient(180deg, #252528 0%, #1a1a1c 100%);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-bottom: 1px solid rgba(0, 0, 0, 0.65);
	}

	.top .digit-num {
		top: 0;
	}

	/* Nửa dưới */
	.bottom {
		bottom: 0;
		border-radius: 0 0 var(--radius) var(--radius);
		background: linear-gradient(180deg, #18181a 0%, #111113 100%);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-top: 1px solid rgba(255, 255, 255, 0.04);
	}

	.bottom .digit-num {
		bottom: 0;
	}

	/* Layering */
	.static-top {
		z-index: 1;
	}

	.static-bottom {
		z-index: 1;
	}

	/* =================================================
	   LÁ LẬT VẬT LÝ 3D (FLIP LEAF)
	   ================================================= */
	.flip-leaf {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 50%;
		z-index: 10;
		transform-origin: bottom center;
		transform-style: preserve-3d;
		-webkit-transform-style: preserve-3d;
		animation: macbook-flap-fall var(--anim-dur) cubic-bezier(0.35, 0.05, 0.25, 1) forwards;
		will-change: transform;
	}

	.leaf-face {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		overflow: hidden;
		backface-visibility: hidden;
		-webkit-backface-visibility: hidden;
		box-sizing: border-box;
	}

	/* Mặt trước: Current Top */
	.leaf-face.front {
		border-radius: var(--radius) var(--radius) 0 0;
		background: linear-gradient(180deg, #252528 0%, #1a1a1c 100%);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-bottom: 1px solid rgba(0, 0, 0, 0.65);
		z-index: 2;
	}

	.leaf-face.front .digit-num {
		top: 0;
	}

	/* Mặt sau: Next Bottom */
	.leaf-face.back {
		transform: rotateX(180deg);
		border-radius: 0 0 var(--radius) var(--radius);
		background: linear-gradient(180deg, #18181a 0%, #111113 100%);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-top: 1px solid rgba(255, 255, 255, 0.04);
		z-index: 1;
	}

	.leaf-face.back .digit-num {
		bottom: 0;
	}

	/* =================================================
	   QUỸ ĐẠO VẬT LÝ & HIỆU ỨNG ÁNH SÁNG MACBOOK
	   ================================================= */
	@keyframes macbook-flap-fall {
		0% {
			transform: rotateX(0deg);
		}
		46% {
			transform: rotateX(-90deg);
		}
		78% {
			transform: rotateX(-182.5deg);
		}
		88% {
			transform: rotateX(-179deg);
		}
		100% {
			transform: rotateX(-180deg);
		}
	}

	/* Lớp phủ bóng đổ động (Lighting & Shadows) */
	.shadow-layer {
		position: absolute;
		inset: 0;
		pointer-events: none;
		opacity: 0;
	}

	.shadow-front {
		background: linear-gradient(180deg, rgba(0, 0, 0, 0.1) 0%, rgba(0, 0, 0, 0.8) 100%);
		animation: front-shadow-darken var(--anim-dur) ease-in forwards;
	}

	@keyframes front-shadow-darken {
		0% { opacity: 0; }
		45% { opacity: 0.75; }
		100% { opacity: 0.75; }
	}

	.shadow-back {
		background: linear-gradient(180deg, rgba(0, 0, 0, 0.8) 0%, rgba(0, 0, 0, 0.1) 100%);
		animation: back-shadow-lighten var(--anim-dur) ease-out forwards;
	}

	@keyframes back-shadow-lighten {
		0% { opacity: 0.8; }
		50% { opacity: 0.6; }
		100% { opacity: 0; }
	}

	.shadow-under.active {
		background: linear-gradient(180deg, rgba(0, 0, 0, 0.65) 0%, rgba(0, 0, 0, 0.1) 100%);
		animation: under-shadow-reveal var(--anim-dur) ease-out forwards;
	}

	@keyframes under-shadow-reveal {
		0% { opacity: 0.7; }
		50% { opacity: 0.3; }
		100% { opacity: 0; }
	}

	.shadow-bottom-ambient.active {
		background: linear-gradient(180deg, rgba(0, 0, 0, 0.8) 0%, rgba(0, 0, 0, 0.2) 100%);
		animation: ambient-shadow-incoming var(--anim-dur) ease-in forwards;
	}

	@keyframes ambient-shadow-incoming {
		0% { opacity: 0; }
		50% { opacity: 0.15; }
		85% { opacity: 0.5; }
		100% { opacity: 0; }
	}

	/* =================================================
	   KHE HỞ TRUNG TÂM VÀ CHỐT BẢN LỀ (HINGES)
	   ================================================= */
	.crease-gap {
		position: absolute;
		top: calc(50% - 0.75px);
		left: 0;
		right: 0;
		height: 1.5px;
		background: #09090b;
		z-index: 25;
		box-shadow: 0 1px 1px rgba(255, 255, 255, 0.05);
	}

	.hinge-notch {
		position: absolute;
		top: 50%;
		width: 4px;
		height: 8px;
		background: #09090b;
		transform: translateY(-50%);
		z-index: 26;
		box-shadow: inset 0 0 2px rgba(255, 255, 255, 0.1);
	}

	.hinge-left {
		left: -1px;
		border-radius: 0 2px 2px 0;
	}

	.hinge-right {
		right: -1px;
		border-radius: 2px 0 0 2px;
	}
</style>
