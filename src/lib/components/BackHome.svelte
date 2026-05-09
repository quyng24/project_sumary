<script lang="ts">
  import { resolve } from '$app/paths';

  let {
    label = 'Back Home'
  }: {
    label?: string;
  } = $props();
</script>

<a class="back-home" href={resolve('/')}>
  <span aria-hidden="true">←</span>
  <span>{label}</span>
</a>

<style>
  .back-home {
    position: fixed;
    top: 1.25rem;
    left: 1.25rem;
    z-index: 20;
    display: inline-flex;
    align-items: center;
    gap: 0.55rem;
    padding: 0.8rem 1.1rem;
    border: 1px solid rgba(255, 255, 255, 0.14);
    border-radius: 999px;
    background:
      linear-gradient(135deg, rgba(255, 255, 255, 0.16), rgba(255, 255, 255, 0.06)),
      rgba(7, 10, 16, 0.7);
    color: #f5f7fb;
    text-decoration: none;
    font-size: 0.95rem;
    font-weight: 700;
    letter-spacing: 0.03em;
    backdrop-filter: blur(18px);
    box-shadow: 0 18px 45px rgba(0, 0, 0, 0.22);
    transition:
      transform 0.2s ease,
      border-color 0.2s ease,
      background-color 0.2s ease;
  }

  .back-home:hover {
    transform: translateY(-2px);
    border-color: rgba(255, 255, 255, 0.26);
    background-color: rgba(255, 255, 255, 0.08);
  }

  .back-home:focus-visible {
    outline: 2px solid #fff;
    outline-offset: 4px;
  }

  @media (max-width: 640px) {
    .back-home {
      top: 0.9rem;
      left: 0.9rem;
      padding: 0.72rem 0.95rem;
      font-size: 0.88rem;
    }
  }
</style>
