<script lang="ts">
  import { onMount } from 'svelte';
  import BackHome from '$lib/components/BackHome.svelte';

  interface DigitState {
    value: string;
    flipping: boolean;
  }

  type DigitKey =
    | 'hourTens'
    | 'hourOnes'
    | 'minuteTens'
    | 'minuteOnes'
    | 'secondTens'
    | 'secondOnes';

  let digits = $state<Record<DigitKey, DigitState>>({
    hourTens:   { value: '0', flipping: false },
    hourOnes:   { value: '0', flipping: false },
    minuteTens: { value: '0', flipping: false },
    minuteOnes: { value: '0', flipping: false },
    secondTens: { value: '0', flipping: false },
    secondOnes: { value: '0', flipping: false },
  });

  function flipDigit(key: DigitKey, newVal: string): void {
    if (digits[key].value !== newVal) {
      digits[key].value = newVal;
      digits[key].flipping = true;

      setTimeout(() => {
        digits[key].flipping = false;
      }, 600);
    }
  }

  function updateClock(): void {
    const now = new Date();
    const hours   = String(now.getHours()).padStart(2, '0');
    const minutes = String(now.getMinutes()).padStart(2, '0');
    const seconds = String(now.getSeconds()).padStart(2, '0');

    flipDigit('hourTens',   hours[0]);
    flipDigit('hourOnes',   hours[1]);
    flipDigit('minuteTens', minutes[0]);
    flipDigit('minuteOnes', minutes[1]);
    flipDigit('secondTens', seconds[0]);
    flipDigit('secondOnes', seconds[1]);
  }

  onMount(() => {
    updateClock();
    const interval = window.setInterval(updateClock, 1000);

    return () => {
      window.clearInterval(interval);
    };
  });
</script>

<BackHome />

<div class="clock-page">
  <div class="flip-clock">
    <div class="flip-unit">
      <div class="digit" class:flip={digits['hourTens'].flipping}>{digits['hourTens'].value}</div>
      <div class="digit" class:flip={digits['hourOnes'].flipping}>{digits['hourOnes'].value}</div>
    </div>
    <div class="separator">:</div>
    <div class="flip-unit">
      <div class="digit" class:flip={digits['minuteTens'].flipping}>{digits['minuteTens'].value}</div>
      <div class="digit" class:flip={digits['minuteOnes'].flipping}>{digits['minuteOnes'].value}</div>
    </div>
    <div class="separator">:</div>
    <div class="flip-unit">
      <div class="digit" class:flip={digits['secondTens'].flipping}>{digits['secondTens'].value}</div>
      <div class="digit" class:flip={digits['secondOnes'].flipping}>{digits['secondOnes'].value}</div>
    </div>
  </div>
</div>

<style>
  :global(body) {
    background-color: #000;
    color: #fff;
    font-family: Arial, Helvetica, sans-serif;
    margin: 0;
  }

  .clock-page {
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    min-height: 100svh;
    padding: 5.5rem 1rem 1rem;
    background-color: #000;
    box-sizing: border-box;
  }

  .flip-clock {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    font-size: 10rem;
  }

  .flip-unit {
    display: flex;
    gap: 0.65rem;
    position: relative;
    perspective: 1000px;
  }

  .digit {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 200px;
    height: 250px;
    line-height: 150px;
    background: #333;
    border-radius: 10px;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.6s;
    box-shadow: 0 20px 45px rgba(0, 0, 0, 0.35);
  }

  .separator {
    font-size: 10rem;
    line-height: 150px;
    margin: 0 0.2rem;
  }

  .digit.flip {
    animation: flip 0.6s;
  }

  @keyframes flip {
    0%   { transform: rotateX(0deg); }
    50%  { transform: rotateX(90deg); }
    100% { transform: rotateX(0deg); }
  }

  /* Mobile */
  @media only screen and (max-width: 46.185em) {
    .clock-page {
      padding: 6rem 0.9rem 1.5rem;
    }

    .flip-clock {
      flex-direction: column;
      gap: 0.75rem;
      font-size: 4.5rem;
    }

    .flip-unit {
      gap: 0.8rem;
    }

    .digit {
      width: 110px;
      height: 132px;
      line-height: 1;
      font-size: 4.5rem;
    }

    .separator {
      font-size: 2.4rem;
      line-height: 1;
      margin: 0;
    }
  }

  /* Tablet */
  @media only screen and (min-width: 46.25em) and (max-width: 63.9375em) {
    .clock-page {
      padding-top: 6.5rem;
    }

    .flip-clock {
      gap: 0.7rem;
      font-size: 6rem;
    }

    .flip-unit {
      gap: 0.75rem;
    }

    .digit {
      width: 130px;
      height: 160px;
      line-height: 1;
      font-size: 6rem;
    }

    .separator {
      font-size: 4.5rem;
      line-height: 1;
      margin: 0 0.1rem;
    }
  }
</style>
