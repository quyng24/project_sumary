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

<div class="clock-page">
	<header class="top-nav">
		<BackHome />
		<div class="title">Mechanical Flip Clock</div>
	</header>

	<main class="clock-content">
		<div class="clock-container">
			<div class="group">
				<Digit value={d[0]} />
				<Digit value={d[1]} />
			</div>

			<div class="separator"></div>

			<div class="group">
				<Digit value={d[2]} />
				<Digit value={d[3]} />
			</div>

			<div class="separator"></div>

			<div class="group">
				<Digit value={d[4]} />
				<Digit value={d[5]} />
			</div>
		</div>
	</main>
</div>

<style>
	.clock-page {
		position: fixed;
		inset: 0;
		background-color: #050505;
		display: flex;
		flex-direction: column;
		color: white;
		font-family: 'Inter', sans-serif;
	}

	.top-nav {
		height: 60px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 1.5rem;
		background: #0a0a0a;
		border-bottom: 1px solid #1a1a1a;
	}

	.title {
		font-size: 0.7rem;
		letter-spacing: 0.3em;
		text-transform: uppercase;
		color: #555;
	}

	.clock-content {
		flex: 1;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.clock-container {
		display: flex;
		align-items: center;
		gap: 2rem; /* Khoảng cách giữa Giờ : Phút : Giây */
	}

	.group {
		display: flex;
		gap: 0.5rem; /* Khoảng cách giữa 2 chữ số trong một cặp */
	}

	.separator {
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
	}

	.separator::before,
	.separator::after {
		content: '';
		width: 10px;
		height: 10px;
		background: #222;
		border-radius: 50%;
	}

	@media (max-width: 850px) {
		.clock-container {
			flex-direction: column;
			gap: 3rem;
		}
		.separator {
			display: none;
		}
	}
</style>