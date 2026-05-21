<script lang="ts">
  import { cn } from "$lib/utils/utils";
  import { onMount } from "svelte";

  const { number = 10 } = $props();

  let meteorStyles = $state<
    {
      top: number;
      left: string;
      animationDelay: string;
      animationDuration: string;
    }[]
  >([]);

  let changeMeteors = (num: number) => {
    const styles = [...new Array(num)].map(() => ({
      top: -20,
      left: Math.floor(Math.random() * 700) + "px",
      animationDelay: (Math.random() * 1 + 0.2) + "s",
      animationDuration: (Math.random() * 8 + 2.9) + "s",
    }));
    meteorStyles = styles;
  };

  onMount(() => {
    changeMeteors(number);
  });

  $effect(() => {
    changeMeteors(number);
  })
</script>

{#each meteorStyles as style, idx (idx)}
  <span
    id="meteor-{idx+1}"
    class={cn(
      "pointer-events-none absolute left-1/2 top-1/2 size-[2.4px] rotate-215 animate-meteor rounded-full bg-slate-500 shadow-[0_0_0_1px_#ffffff10]"
    )}
    style="top: {style.top}px; left: {style.left}; animation-delay: {style.animationDelay}; animation-duration: {style.animationDuration};"
  >
    <div class="pointer-events-none absolute top-1/2 -z-10 h-px w-200 -translate-y-1/2 bg-linear-to-r from-slate-500 via-blue-600/30 to-transparent"></div>
  </span>
{/each}
