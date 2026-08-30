<script lang="ts">
  import { resolve } from '$app/paths';

  const projects = [
    { title: 'Flip Clock', href: resolve('/clock'), tag: 'Motion', tagType: 'motion', summary: 'Đồng hồ số phong cách cổ điển với hiệu ứng lật trang (flip). Tập trung xử lý tính toán thời gian thực và tối ưu chuyển động.' },
    { title: 'Animate Planet', href: resolve('/planet'), tag: 'SVG Animation', tagType: 'motion', summary: 'Mô phỏng hệ hành lang quỹ đạo bằng CSS Animation và SVG. Minh họa cách điều phối các chuyển động phức tạp.' },
    { title: 'Christmas Tree', href: resolve('/christmastree'), tag: 'Experimental', tagType: 'motion', summary: 'Ứng dụng diễn họa cây thông Noel bằng mã nhị phân và hiệu ứng ánh sáng mùa lễ hội.' },
    { title: 'Heart Particle', href: resolve('/tym'), tag: 'Canvas', tagType: 'motion', summary: 'Hiệu ứng trái tim rực rỡ tạo bởi hàng trăm hạt particle toán học. Mô phỏng vật lý gia tốc trên Canvas 2D.' },
    { title: 'Game Tetris', href: resolve('/tetris'), tag: 'Game', tagType: 'game', summary: 'Trò chơi xếp gạch kinh điển. Thách thức logic xử lý mảng hai chiều và tính toán va chạm.' },
    { title: 'Game Tictactoe', href: resolve('/tictactoe'), tag: 'Game', tagType: 'game', summary: 'Trò chơi Caro đối kháng với giao diện Neon hiện đại. Tích hợp thuật toán phán đoán nước đi.' },
    { title: 'Game Platformer', href: resolve('/platformer'), tag: 'Game', tagType: 'game', summary: 'Game đi ải 2D với trọng lực và va chạm vật lý. Trọng tâm vào Game Loop và camera bám đuổi.' },
    { title: 'Excel Web', href: resolve('/excel'), tag: 'App', tagType: 'app', summary: 'Bảng tính trực tuyến hỗ trợ công thức toán học. Trình diễn kỹ thuật quản lý dữ liệu hiệu suất cao.' }
  ];

  const meteors = [
    { t: '10%', l: '80%', d: '0s', du: '6s' },
    { t: '30%', l: '90%', d: '2s', du: '8s' },
    { t: '0%', l: '50%', d: '4s', du: '7s' },
    { t: '15%', l: '100%', d: '6s', du: '10s' },
    { t: '50%', l: '110%', d: '1s', du: '9s' }
  ];
</script>

<svelte:head>
  <title>Portfolio | Svelte Showcase</title>
</svelte:head>

<div class="app-shell">
  <div class="night-sky" aria-hidden="true">
    <div class="stars-static"></div>
    {#each meteors as m, i (i)}
      <div 
        class="meteor" 
        style="top: {m.t}; left: {m.l}; --delay: {m.d}; --duration: {m.du};"
      ></div>
    {/each}
  </div>

  <header class="navbar">
    <div class="nav-content">
      <span class="logo">SVELTE.LAB</span>
      <nav>
        <a href="https://github.com/quyng24" target="_blank">Github</a>
      </nav>
    </div>
  </header>

  <main class="main-content">
    <section class="hero">
      <div class="status-badge">
        <span class="dot"></span> Available for experiments
      </div>
      <h1>Collection of <br/><span class="highlight">Interface Experiments</span></h1>
      <p class="subtitle">Tổng hợp các dự án thử nghiệm về giao diện, logic game và ứng dụng web được xây dựng trên nền tảng Svelte 5.</p>
    </section>

    <div class="project-list">
      {#each projects as project, i (project.title)}
        <a href={project.href} class="project-row">
          <span class="index">{(i + 1).toString().padStart(2, '0')}</span>
          <h3>{project.title}</h3>
          <div class="tag-box">{project.tag}</div>
          <p class="summary">{project.summary}</p>
          <span class="arrow-icon">→</span>
        </a>
      {/each}
    </div>
  </main>

  <footer class="site-footer">
    <div class="footer-container">
      <div class="footer-left">
        <span class="brand-status">
          <span class="status-dot"></span>
          SYSTEM OPERATIONAL
        </span>
        <p class="copyright">© 2024 SVELTE.LAB — BUILT BY NGUYEN QUY CODER</p>
      </div>

      <div class="footer-center">
        <div class="tech-stack">
          <span>SVELTE 5</span>
          <span class="separator">/</span>
          <span>SVELTEKIT</span>
          <span class="separator">/</span>
          <span>VITE</span>
        </div>
      </div>

      <div class="footer-right">
        <nav class="footer-nav">
          <a href="https://github.com/quyng24" target="_blank">GITHUB</a>
          <a href="mailto:quy24.dev@gmail.com">CONTACT</a>
        </nav>
      </div>
    </div>
  </footer>
</div>

<style>
  :global(:root) {
    --bg-color: #000000;
    --border-color: #222222;
    --text-primary: #ffffff;
    --text-secondary: #888888;
  }

  :global(body) {
    margin: 0;
    padding: 0;
    background-color: #000; 
    color: #fff;
    overflow-x: hidden;
    font-family: 'Inter', -apple-system, sans-serif;
  }

  .app-shell {
    position: relative;
    min-height: 100vh;
    z-index: 1;
  }

  .night-sky {
    position: fixed;
    inset: 0;
    z-index: -1;
    background: radial-gradient(ellipse at bottom, #1B2735 0%, #090A0F 100%);
    overflow: hidden;
  }

  .stars-static {
    position: absolute;
    width: 2px;
    height: 2px;
    background: transparent;
    box-shadow: 14vw 15vh #fff, 25vw 35vh #fff, 45vw 10vh #fff, 75vw 40vh #fff, 85vw 15vh #fff, 10vw 60vh #fff, 30vw 80vh #fff, 60vw 90vh #fff, 90vw 70vh #fff;
    opacity: 0.3;
  }

  .meteor {
    position: absolute;
    width: 2px;
    height: 2px;
    background: #fff;
    border-radius: 50%;
    box-shadow: 0 0 0 4px rgba(255, 255, 255, 0.1), 0 0 0 8px rgba(255, 255, 255, 0.1), 0 0 20px rgba(255, 255, 255, 1);
    animation: meteor-fall var(--duration) linear var(--delay) infinite;
    opacity: 0;
    will-change: transform;
  }

  .meteor::after {
    content: "";
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    width: 150px;
    height: 1px;
    background: linear-gradient(90deg, #fff, transparent);
  }

  @keyframes meteor-fall {
    0% {
      transform: rotate(315deg) translateX(0);
      opacity: 0;
    }
    10% {
      opacity: 1;
    }
    30% {
      transform: rotate(315deg) translateX(-1000px);
      opacity: 0;
    }
    100% {
      transform: rotate(315deg) translateX(-1000px);
      opacity: 0;
    }
  }

  .navbar {
    border-bottom: 1px solid var(--border-color);
    padding: 1.25rem 0;
    background: rgba(0,0,0,0.5);
    backdrop-filter: blur(10px);
  }

  .nav-content {
    max-width: 1100px;
    margin: 0 auto;
    padding: 0 1.5rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .logo { font-weight: 800; letter-spacing: -0.5px; font-size: 1.1rem; }
  nav a { color: var(--text-secondary); text-decoration: none; font-size: 0.9rem; font-weight: 500; }

  .main-content {
    max-width: 1100px;
    margin: 0 auto;
    padding: 6rem 1.5rem;
  }

  .hero { margin-bottom: 5rem; position: relative; }

  h1 {
    font-size: clamp(2.5rem, 6vw, 4rem);
    font-weight: 800;
    line-height: 1.1;
    margin: 0 0 1.5rem;
    letter-spacing: -0.03em;
  }

  .highlight { color: var(--text-secondary); }

  .subtitle {
    font-size: 1.1rem;
    color: var(--text-secondary);
    max-width: 600px;
    line-height: 1.6;
  }

  .project-list {
    display: flex;
    flex-direction: column;
    border-top: 1px solid #222;
  }

  .project-row {
    display: grid;
    grid-template-columns: 60px 1.5fr 140px 2fr 40px;
    align-items: center;
    padding: 2rem 1rem;
    text-decoration: none;
    color: #888;
    border-bottom: 1px solid #222;
    transition: all 0.3s ease;
    background: rgba(0,0,0,0.2);
  }

  .project-row:hover {
    color: #fff;
    background: rgba(255, 255, 255, 0.05);
    padding-left: 1.5rem;
  }

  .index { font-family: 'JetBrains Mono', monospace; opacity: 0.4; }
  h3 { margin: 0; font-size: 1.4rem; color: #eee; }
  .project-row:hover h3 { color: #3b82f6; }

  .tag-box {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.7rem;
    border: 1px solid #333;
    padding: 4px 8px;
    border-radius: 4px;
    width: fit-content;
  }

  .summary {
    font-size: 0.9rem;
    line-height: 1.5;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
    padding-right: 1rem;
  }

  .arrow-icon { font-size: 1.2rem; opacity: 0; transition: 0.3s; }
  .project-row:hover .arrow-icon { opacity: 1; transform: translateX(5px); }

  .site-footer {
    margin-top: 8rem;
    padding: 4rem 0;
    border-top: 1px solid var(--border-color);
    background: rgba(0, 0, 0, 0.3);
    backdrop-filter: blur(10px);
  }

  .footer-container {
    max-width: 1100px;
    margin: 0 auto;
    padding: 0 1.5rem;
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    gap: 2rem;
  }

  .brand-status {
    display: flex;
    align-items: center;
    gap: 8px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.7rem;
    color: #10b981;
    letter-spacing: 1px;
    margin-bottom: 0.8rem;
  }

  .status-dot {
    width: 4px;
    height: 4px;
    background: #10b981;
    border-radius: 50%;
    box-shadow: 0 0 8px #10b981;
  }

  .copyright {
    margin: 0;
    font-size: 0.75rem;
    color: #444;
    letter-spacing: 0.5px;
  }

  .tech-stack {
    display: flex;
    gap: 12px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.65rem;
    color: #333;
    letter-spacing: 2px;
  }

  .tech-stack .separator {
    color: #222;
  }

  .footer-nav {
    display: flex;
    gap: 2rem;
  }

  .footer-nav a {
    text-decoration: none;
    color: #666;
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 1.5px;
    transition: color 0.3s ease;
  }

  .footer-nav a:hover {
    color: var(--accent);
  }

  @media (max-width: 1024px) {
    .project-row { grid-template-columns: 40px 1fr 120px; gap: 1rem; }
    .summary, .tag-box { display: none; }
  }

  @media (max-width: 850px) {
    .footer-container {
      flex-direction: column;
      align-items: center;
      text-align: center;
      gap: 3rem;
    }
    
    .footer-center {
      order: 3;
    }
    
    .footer-right {
      order: 2;
    }
  }

  @media (max-width: 640px) {
    .project-row { display: flex; flex-direction: column; align-items: flex-start; }
    .main-content { padding-top: 3rem; }
    .meteor::after { width: 80px; }
  }
</style>
