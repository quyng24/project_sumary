<svelte:options runes={true} />

<script lang="ts">
	import { onMount } from 'svelte';
    import BackHome from '$lib/components/BackHome.svelte';

	const RX = 220;
	const RY = 80;
	const PERIOD = 6000;

	let angle = $state(0);
	let sx = $derived(Math.cos(angle) * RX);
	let sy = $derived(Math.sin(angle) * RY);
	let isFront = $derived(sy <= 0);
	let shadowOffset = $derived(-Math.cos(angle) * 110);

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

<div class="wrapper">
    <BackHome/>
	<div class="scene">
		<svg class="orbit-svg" viewBox="-240 -100 480 200" xmlns="http://www.w3.org/2000/svg">
			<ellipse cx="0" cy="0" rx={RX} ry={RY}
				fill="none"
				stroke="rgba(255,255,255,0.06)"
				stroke-width="1"
				stroke-dasharray="6 8"
			/>
		</svg>

		{#if !isFront}
			<div class="satellite"
				style="transform: translate(calc(-50% + {sx}px), calc(-50% + {sy}px));">
				<div class="satellite-shadow"
					style="transform: translateX({shadowOffset}%);"></div>
			</div>
		{/if}

		<div class="planet">
			<div class="planet-shine"></div>
			<div class="planet-crater crater-1"></div>
			<div class="planet-crater crater-2"></div>
			<div class="planet-crater crater-3"></div>
		</div>

		{#if isFront}
			<div class="satellite"
				style="transform: translate(calc(-50% + {sx}px), calc(-50% + {sy}px));">
				<div class="satellite-shadow"
					style="transform: translateX({shadowOffset}%);"></div>
			</div>
		{/if}
	</div>
</div>

<style>
	:global(body) {
		margin: 0;
		background: #101227;
		font-size: 12px;
		font-family: Arial, Helvetica, sans-serif;
	}

	.wrapper {
		width: 100%;
		min-height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		background: #101227;
		overflow: hidden;
	}

	.scene {
		position: relative;
		width: 30em;
		height: 30em;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.orbit-svg {
		position: absolute;
		width: 480px;
		height: 200px;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
		pointer-events: none;
		overflow: visible;
	}

	.planet {
		position: absolute;
		width: 30em;
		height: 30em;
		border-radius: 50%;
		background: radial-gradient(circle at 38% 35%, #ffffff 0%, #d8d8d0 55%, #a8a8a0 100%);
		box-shadow:
			0 0 5em 0 #fed84c80,
			0 0 20em 4em #e8a55233,
			0 0 55em 8em #ff4d4d1a;
		overflow: hidden;
	}

	.planet-shine {
		position: absolute;
		top: 8%; left: 12%;
		width: 35%; height: 30%;
		border-radius: 50%;
		background: radial-gradient(circle, rgba(255,255,255,0.65) 0%, transparent 70%);
		filter: blur(6px);
	}

	.planet-crater {
		position: absolute;
		border-radius: 50%;
		background: rgba(140,130,120,0.22);
		border: 1px solid rgba(100,90,80,0.12);
	}
	.crater-1 { width: 22%; height: 22%; top: 55%; left: 20%; }
	.crater-2 { width: 14%; height: 14%; top: 35%; left: 60%; }
	.crater-3 { width: 10%; height: 10%; top: 65%; left: 58%; }

	.satellite {
		position: absolute;
		top: 50%;
		left: 50%;
		width: 5em;
		height: 5em;
		border-radius: 50%;
		background: radial-gradient(circle at 40% 35%, #fff5e4 0%, #ffedd1 50%, #e8c89a 100%);
		overflow: hidden;
		box-shadow: 0 0 1.5em 0.5em rgba(255, 220, 150, 0.15);
	}

	.satellite-shadow {
		position: absolute;
		inset: 0;
		border-radius: 50%;
		background: #1e2138;
		box-shadow: 0 0 0.75em 0.95em #1e2138;
	}

	@media only screen and (max-width: 46.1875em) {
		.scene  { width: 10em; height: 10em; }
		.planet { width: 10em; height: 10em; }
		.satellite { width: 3em; height: 3em; }
		.orbit-svg { width: 260px; height: 120px; }
	}

	@media only screen and (min-width: 46.25em) and (max-width: 63.9375em) {
		.scene  { width: 15em; height: 15em; }
		.planet { width: 15em; height: 15em; }
		.satellite { width: 4em; height: 4em; }
		.orbit-svg { width: 360px; height: 160px; }
	}
</style>