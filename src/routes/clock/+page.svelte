<script lang="ts">
	import { onMount } from 'svelte';
	import BackHome from '$lib/components/BackHome.svelte';
	import Digit from '$lib/components/Digit.svelte';

	let time = $state(new Date());

	onMount(() => {
		const interval = setInterval(() => {
			time = new Date();
		}, 1000);
		return () => clearInterval(interval);
	});

	const getDigits = (t: Date) => {
		const h = String(t.getHours()).padStart(2, '0');
		const m = String(t.getMinutes()).padStart(2, '0');
		const s = String(t.getSeconds()).padStart(2, '0');
		return [h[0], h[1], m[0], m[1], s[0], s[1]];
	};

	let d = $derived(getDigits(time));
</script>

<svelte:head>
	<title>Mechanical Flip Clock | Svelte 5</title>
	<meta
		name="description"
		content="Đồng hồ số lật phong cách Split-flap cơ khí tối giản với hiệu ứng chuyển động vật lý mượt mà."
	/>
</svelte:head>

<div class="clock-page">
	<!-- Top Navigation (BackHome không che tiêu đề) -->
	<header class="top-nav">
		<div class="nav-left">
			<BackHome label="Back Home" />
		</div>
		<div class="title">Mechanical Flip Clock</div>
	</header>

	<!-- Main Clock Content (Không có card bọc ngoài, hiển thị số thuần khiết) -->
	<main class="clock-content">
		<div class="clock-container">
			<!-- GIỜ -->
			<div class="group">
				<Digit value={d[0]} />
				<Digit value={d[1]} />
			</div>

			<!-- PHÂN CÁCH 1 -->
			<div class="separator" aria-hidden="true"></div>

			<!-- PHÚT -->
			<div class="group">
				<Digit value={d[2]} />
				<Digit value={d[3]} />
			</div>

			<!-- PHÂN CÁCH 2 -->
			<div class="separator" aria-hidden="true"></div>

			<!-- GIÂY -->
			<div class="group">
				<Digit value={d[4]} />
				<Digit value={d[5]} />
			</div>
		</div>
	</main>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@500;700&family=Inter:wght@500;600;700&display=swap');

	.clock-page {
		position: fixed;
		inset: 0;
		background-color: #050505;
		display: flex;
		flex-direction: column;
		color: #ffffff;
		font-family: 'Inter', -apple-system, sans-serif;
		overflow: hidden;
		user-select: none;
	}

	/* =================================================
	   TOP NAV (Khắc phục lỗi nút BackHome che tiêu đề)
	   ================================================= */
	.top-nav {
		height: 60px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 1.5rem;
		background: #0a0a0a;
		border-bottom: 1px solid #1a1a1a;
		z-index: 50;
		box-sizing: border-box;
	}

	.nav-left {
		display: flex;
		align-items: center;
	}

	/* Ép BackHome về vị trí static trong nav-left, không dùng fixed để tránh đè tiêu đề */
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
		box-shadow: none !important;
	}

	:global(.top-nav .back-home:hover) {
		background: rgba(255, 255, 255, 0.12) !important;
		border-color: rgba(255, 255, 255, 0.22) !important;
	}

	.title {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.75rem;
		letter-spacing: 0.25em;
		text-transform: uppercase;
		color: #666;
		white-space: nowrap;
	}

	/* =================================================
	   CLOCK CONTENT (Không bọc card ngoài)
	   ================================================= */
	.clock-content {
		flex: 1;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 1.5rem;
		box-sizing: border-box;
	}

	.clock-container {
		display: flex;
		align-items: center;
		justify-content: center;
		gap: clamp(1rem, 2.5vw, 2.2rem);
	}

	.group {
		display: flex;
		gap: clamp(0.3rem, 0.8vw, 0.6rem);
	}

	/* Dấu hai chấm phân cách thanh lịch */
	.separator {
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
	}

	.separator::before,
	.separator::after {
		content: '';
		width: clamp(6px, 1.2vw, 10px);
		height: clamp(6px, 1.2vw, 10px);
		background: rgba(255, 255, 255, 0.22);
		border-radius: 50%;
		box-shadow: 0 0 8px rgba(255, 255, 255, 0.08);
	}

	/* =================================================
	   RESPONSIVE
	   ================================================= */
	@media (max-width: 640px) {
		.top-nav {
			padding: 0 1rem;
		}

		.title {
			font-size: 0.68rem;
			letter-spacing: 0.18em;
		}

		.clock-content {
			padding: 1rem 0.5rem;
		}

		.clock-container {
			gap: 0.6rem;
		}

		.group {
			gap: 0.25rem;
		}

		.separator {
			gap: 1rem;
		}
	}

	@media (max-width: 380px) {
		.clock-container {
			gap: 0.4rem;
		}

		.group {
			gap: 0.2rem;
		}
	}
</style>
