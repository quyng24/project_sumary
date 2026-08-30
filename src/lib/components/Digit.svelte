<script lang="ts">
	let { value } = $props();

	// Giá trị đang hiển thị hoàn chỉnh
	let currentValue = $state(value);

	// Trạng thái animation
	let isFlipping = $state(false);

	/*
	 * Khi value từ component cha thay đổi,
	 * bắt đầu animation.
	 */
	$effect(() => {
		if (value === currentValue) return;

		if (!isFlipping) {
			isFlipping = true;
		}
	});

	/*
	 * Animation kết thúc.
	 *
	 * Lúc này toàn bộ card NEXT đã hiển thị,
	 * nên chỉ cần chuyển currentValue sang value.
	 */
	function handleAnimationEnd() {
		currentValue = value;
		isFlipping = false;
	}
</script>

<div class="digit-container">
	<!-- ==========================================
	     CURRENT
	     ========================================== -->

	<!-- Nửa trên của số hiện tại -->
	<div class="card top current-top">
		<span>{currentValue}</span>
	</div>

	<!-- Nửa dưới của số hiện tại -->
	<div class="card bottom current-bottom">
		<span>{currentValue}</span>
	</div>

	<!-- ==========================================
	     NEXT
	     ========================================== -->

	<!-- Nửa trên của số tiếp theo -->
	<div class="card top next-top">
		<span>{value}</span>
	</div>

	<!-- Nửa dưới của số tiếp theo -->
	<div class="card bottom next-bottom">
		<span>{value}</span>
	</div>

	<!-- ==========================================
	     FLIP LEAF
	     ========================================== -->

	<div
		class="flip-leaf"
		class:flipping={isFlipping}
		onanimationend={handleAnimationEnd}
	>
		<!-- Mặt trước: current TOP -->
		<div class="leaf-face leaf-front">
			<span>{currentValue}</span>
		</div>

		<!-- Mặt sau: next BOTTOM -->
		<div class="leaf-face leaf-back">
			<span>{value}</span>
		</div>
	</div>

	<!-- Đường giữa -->
	<div class="center-line"></div>
</div>

<style>
	.digit-container {
		--w: clamp(70px, 15vw, 140px);
		--h: calc(var(--w) * 1.4);
		--radius: 8px;
		--anim-speed: 0.6s;

		position: relative;

		width: var(--w);
		height: var(--h);

		background: #111;

		border-radius: var(--radius);

		font-size: calc(var(--w) * 1.1);
		font-weight: 700;

		line-height: var(--h);

		font-family: 'JetBrains Mono', monospace;

		perspective: 1000px;

		overflow: hidden;
	}

	/* =================================================
	   CARD BASE
	   ================================================= */

	.card {
		position: absolute;

		left: 0;

		width: 100%;
		height: 50%;

		overflow: hidden;

		background: #1a1a1a;

		backface-visibility: hidden;
	}

	.card span,
	.flip-leaf span {
		position: absolute;

		left: 50%;

		width: 100%;

		text-align: center;

		transform: translateX(-50%);
	}

	/* =================================================
	   TOP
	   ================================================= */

	.top {
		top: 0;

		border-radius: var(--radius) var(--radius) 0 0;

		border-bottom: 1px solid rgba(0, 0, 0, 0.5);
	}

	.top span {
		top: 0;
	}

	/* =================================================
	   BOTTOM
	   ================================================= */

	.bottom {
		bottom: 0;

		border-radius: 0 0 var(--radius) var(--radius);
	}

	.bottom span {
		bottom: 0;
	}

	/* =================================================
	   LAYER ORDER
	   ================================================= */

	/*
	 * CURRENT là nền.
	 */
	.current-top {
		z-index: 1;
	}

	.current-bottom {
		z-index: 1;
	}

	/*
	 * NEXT nằm phía dưới.
	 *
	 * Chúng ta sẽ cho NEXT xuất hiện
	 * thông qua animation.
	 */
	.next-top {
		z-index: 0;
	}

	.next-bottom {
		z-index: 0;
	}

	/* =================================================
	   FLIP LEAF
	   ================================================= */

	.flip-leaf {
		position: absolute;

		top: 0;
		left: 0;

		width: 100%;
		height: 50%;

		z-index: 5;

		transform-origin: bottom center;

		transform-style: preserve-3d;

		transform: rotateX(0deg);
	}

	.leaf-face {
		position: absolute;

		inset: 0;

		width: 100%;
		height: 100%;

		overflow: hidden;

		background: #1a1a1a;

		backface-visibility: hidden;

		transform-style: preserve-3d;
	}

	/* Mặt trước */
	.leaf-front {
		border-radius: var(--radius) var(--radius) 0 0;

		z-index: 2;

		box-shadow:
			inset 0 -10px 30px rgba(0, 0, 0, 0.15);
	}

	.leaf-front span {
		top: 0;
	}

	/* Mặt sau */
	.leaf-back {
		transform: rotateX(180deg);

		border-radius: 0 0 var(--radius) var(--radius);
	}

	.leaf-back span {
		bottom: 0;
	}

	/* =================================================
	   ANIMATION
	   ================================================= */

	.flip-leaf.flipping {
		animation:
			flip-down var(--anim-speed)
			cubic-bezier(0.4, 0, 0.2, 1)
			forwards;
	}

	@keyframes flip-down {
		0% {
			transform: rotateX(0deg);
		}

		100% {
			transform: rotateX(-180deg);
		}
	}

	/* =================================================
	   NEXT LAYER
	   ================================================= */

	/*
	 * Khi bắt đầu flip:
	 *
	 * NEXT TOP phải nằm phía sau leaf.
	 *
	 * Sau khi leaf đi qua 90deg,
	 * mặt sau của leaf sẽ che/hiện đúng phần cần thiết.
	 */

	.flipping .next-top {
		z-index: 2;
	}

	.flipping .next-bottom {
		z-index: 1;
	}

	/* =================================================
	   CURRENT LAYER
	   ================================================= */

	.flipping .current-bottom {
		z-index: 1;
	}

	.flipping .current-top {
		z-index: 0;
	}

	/* =================================================
	   CENTER LINE
	   ================================================= */

	.center-line {
		position: absolute;

		top: 50%;
		left: 0;

		width: 100%;
		height: 1px;

		background: rgba(0, 0, 0, 0.7);

		z-index: 20;
	}
</style>