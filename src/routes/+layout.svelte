<script lang="ts">
	import { afterNavigate } from '$app/navigation';
	import { onMount } from 'svelte';
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();

	afterNavigate(() => {
		if (typeof window === 'undefined') return;

		window.scrollTo(0, 0);
		document.documentElement.scrollTop = 0;
		document.body.scrollTop = 0;
	});

	onMount(() => {
		if ('scrollRestoration' in history) {
			history.scrollRestoration = 'manual';
		}

		return () => {
			if ('scrollRestoration' in history) {
				history.scrollRestoration = 'auto';
			}
		};
	});
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
{@render children()}
